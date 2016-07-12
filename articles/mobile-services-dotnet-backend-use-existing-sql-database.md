<properties 
	pageTitle="使用现有的 SQL 数据库和移动服务 .NET 后端生成服务 | Azure" 
	description="了解如何将现有的云或本地 SQL 数据库与基于 .NET 的移动服务结合使用" 
	services="mobile-services" 
	documentationCenter="" 
	authors="ggailey777" 
	manager="dwrede" 
	editor="mollybos"/>

<tags 
	ms.service="mobile-services" 
	ms.date="11/09/2015"
	wacn.date="01/29/2016"/>


# 使用现有的 SQL 数据库和移动服务 .NET 后端生成服务

移动服务 .NET 后端可方便你利用现有的资产来生成移动服务。（在本地或云中）使用可能已被其他应用程序使用的现有 SQL 数据库让现有数据可供移动客户端使用是特别有趣的方案之一。在此情况下，数据库模型（或*架构*）必须保持不变才能使现有方案继续工作。

<a name="ExistingModel"></a>
## 探索现有的数据库模型

在本教程中，我们将使用以你的移动服务创建的数据库，但不使用创建的默认模型。我们将手动创建任意模型，以代表你可能具有的现有应用程序。有关如何改为连接到本地数据库的完整详细信息，请查看[使用混合连接从 Azure 移动服务连接到本地 SQL Server](/documentation/articles/mobile-services-dotnet-backend-hybrid-connections-get-started/)。

1. 首先，请在 **Visual Studio 2013 Update 2** 中创建移动服务服务器项目，或使用可在 [Azure 管理门户](http://manage.windowsazure.cn)中从服务的“移动服务”选项卡下载的快速入门项目。对于本教程，我们假设你的服务器项目名称为 **ShoppingService**。

2. 在 **Models** 文件夹中创建 **Customer.cs** 文件，并使用以下实现。需要将 **System.ComponentModel.DataAnnotations** 的程序集引用添加到项目中。

        using System.Collections.Generic;
        using System.ComponentModel.DataAnnotations;

        namespace ShoppingService.Models
        {
            public class Customer
            {
                [Key]
                public int CustomerId { get; set; }
                
                public string Name { get; set; }

                public virtual ICollection<Order> Orders { get; set; }

            }
        }

3. 在 **Models** 文件夹中创建 **Order.cs** 文件，并使用以下实现：
    
        using System.ComponentModel.DataAnnotations;

        namespace ShoppingService.Models
        {
            public class Order
            {
                [Key]
                public int OrderId { get; set; }

                public string Item { get; set; }

                public int Quantity { get; set; }

                public bool Completed { get; set; }

                public int CustomerId { get; set; }
              
                public virtual Customer Customer { get; set; }

            }
        }

    你将会注意到，这两个类存在某种*关系*：每个**订单**与一个**客户**关联，而一个**客户**可与多个**订单**关联。互有关系在现有数据模型中很常见。

4. 在 **Models** 文件夹中创建 **ExistingContext.cs** 文件，并按如下所示实现它：

        using System.Data.Entity;

        namespace ShoppingService.Models
        {
            public class ExistingContext : DbContext
            {
                public ExistingContext()
                    : base("Name=MS_TableConnectionString")
                {
                }

                public DbSet<Customer> Customers { get; set; }

                public DbSet<Order> Orders { get; set; }

            }
        }

上述结构很接近你可能已用于现有应用程序的现有 Entity Framework 模型。请注意，在此阶段，模型无法以任何方式识别移动服务。

<a name="DTOs"></a>
## 为移动服务创建数据传输对象 (DTO)

你想要在移动服务中使用的数据模型可能很复杂；其中可能包含数百个具有各种关系的实体。在生成移动应用程序时，我们通常会简化数据模型并消除关系（或手动处理关系），以尽可能减少在应用程序与服务之间来回发送的负载。在本部分中，我们将创建一组简化的对象（称为“数据传输对象”，缩写为“DTO”），并将其映射到你在数据库中拥有的数据，但仅包含移动应用程序所需的最少属性集。

1. 在服务项目的 **DataObjects** 文件夹中创建 **MobileCustomer.cs** 文件，并使用以下实现：

        using Microsoft.WindowsAzure.Mobile.Service;

        namespace ShoppingService.DataObjects
        {
            public class MobileCustomer : EntityData
            {
                public string Name { get; set; }
            }
        }

    请注意，此类与模型中的 **Customer** 类相类似，差别在于 **Order** 的关系属性已删除。若要使对象正常使用移动服务脱机同步，需要使用一组*系统属性*来进行乐观并发，因此你会发现，DTO 继承自包含这些属性的 [**EntityData**](http://msdn.microsoft.com/library/microsoft.windowsazure.mobile.service.entitydata.aspx)。原始模型中的基于 int 的 **CustomerId** 属性将替换为 **EntityData** 中基于字符串的 **Id** 属性，这将是移动服务使用的 **Id**。

2. 在服务项目的 **DataObjects** 文件夹中创建 **MobileOrder.cs** 文件。

        using Microsoft.WindowsAzure.Mobile.Service;
        using Newtonsoft.Json;
        using System.ComponentModel.DataAnnotations;
        using System.ComponentModel.DataAnnotations.Schema;

        namespace ShoppingService.DataObjects
        {
            public class MobileOrder : EntityData
            {
                public string Item { get; set; }

                public int Quantity { get; set; }

                public bool Completed { get; set; }

                [NotMapped]
                public int CustomerId { get; set; }

                [Required]
                public string MobileCustomerId { get; set; }

                public string MobileCustomerName { get; set; }
            }
        }

    **Customer** 关系属性已替换为 **Customer** 名称，以及可用来为客户端上的关系手动建模的 **MobileCustomerId** 属性。现在你可以忽略 **CustomerId** 属性，以后才会使用此属性。

3. 你可能会发现，在 **EntityData** 基类上添加系统属性后，DTO 的属性数目比模型类型还多。显然，需要一个位置来存储这些属性，因此我们将在原始数据库中额外添加一些列。虽然这样会更改数据库，但并不会中断现有的应用程序，因为这些更改只是附加的（将新的列添加到架构）。为此，请将以下语句添加到 **Customer.cs** 和 **Order.cs** 的顶部：
    
        using System.ComponentModel.DataAnnotations.Schema;
        using Microsoft.WindowsAzure.Mobile.Service.Tables;
        using System;

4. 然后，将这些附加属性添加到每个类：

        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Index]
        [TableColumn(TableColumnType.CreatedAt)]
        public DateTimeOffset? CreatedAt { get; set; }

        [TableColumn(TableColumnType.Deleted)]
        public bool Deleted { get; set; }

        [Index]
        [TableColumn(TableColumnType.Id)]
        [MaxLength(36)]
        public string Id { get; set; }

        [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
        [TableColumn(TableColumnType.UpdatedAt)]
        public DateTimeOffset? UpdatedAt { get; set; }

        [TableColumn(TableColumnType.Version)]
        [Timestamp]
        public byte[] Version { get; set; }

5. 刚刚添加的系统属性具有某些会在数据库操作期间明确发生的内置行为（例如，自动更新创建/更新时间）。若要启用这些行为，我们需要更改 **ExistingContext.cs**。在该文件的顶部，添加以下代码：
    
        using System.Data.Entity.ModelConfiguration.Conventions;
        using Microsoft.WindowsAzure.Mobile.Service.Tables;
        using System.Linq;

6. 然后，在 **ExistingContext** 的正文中，重写 [**OnModelCreating**](http://msdn.microsoft.com/zh-cn/library/system.data.entity.dbcontext.onmodelcreating.aspx)：

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Conventions.Add(
                new AttributeToColumnAnnotationConvention<TableColumnAttribute, string>(
                    "ServiceTableColumn", (property, attributes) => attributes.Single().ColumnType.ToString()));

            base.OnModelCreating(modelBuilder);
        } 

7. 让我们在数据库中填充一些示例数据。打开文件 **WebApiConfig.cs**。创建新的 [**IDatabaseInitializer**](http://msdn.microsoft.com/zh-cn/library/gg696323.aspx)，并按如下所示在 **Register** 方法中对它进行配置。

        using Microsoft.WindowsAzure.Mobile.Service;
        using ShoppingService.Models;
        using System;
        using System.Collections.Generic;
        using System.Collections.ObjectModel;
        using System.Data.Entity;
        using System.Web.Http;

        namespace ShoppingService
        {
            public static class WebApiConfig
            {
                public static void Register()
                {
                    ConfigOptions options = new ConfigOptions();

                    HttpConfiguration config = ServiceConfig.Initialize(new ConfigBuilder(options));

                    Database.SetInitializer(new ExistingInitializer());
                }
            }

            public class ExistingInitializer : ClearDatabaseSchemaIfModelChanges<ExistingContext>
            {
                protected override void Seed(ExistingContext context)
                {
                    List<Order> orders = new List<Order>
                    {
                        new Order { OrderId = 10, Item = "Guitars", Quantity = 2, Id = Guid.NewGuid().ToString()},
                        new Order { OrderId = 20, Item = "Drums", Quantity = 10, Id = Guid.NewGuid().ToString()},
                        new Order { OrderId = 30, Item = "Tambourines", Quantity = 20, Id = Guid.NewGuid().ToString() }
                    };

                    List<Customer> customers = new List<Customer>
                    {
                        new Customer { CustomerId = 1, Name = "John", Orders = new Collection<Order> { 
                            orders[0]}, Id = Guid.NewGuid().ToString()},
                        new Customer { CustomerId = 2, Name = "Paul", Orders = new Collection<Order> { 
                            orders[1]}, Id = Guid.NewGuid().ToString()},
                        new Customer { CustomerId = 3, Name = "Ringo", Orders = new Collection<Order> { 
                            orders[2]}, Id = Guid.NewGuid().ToString()},
                    };

                    foreach (Customer c in customers)
                    {
                        context.Customers.Add(c);
                    }

                    base.Seed(context);
                }
            }
        }

<a name="Mapping"></a>
## 在 DTO 与模型之间创建映射

现在我们已有模型类型 **Customer** 和 **Order** 以及 DTO **MobileCustomer** 和 **MobileOrder**，但我们需要指示后端自动在两者之间转换。此时，移动服务依赖于 [**AutoMapper**](http://automapper.org/)，这是已在项目中引用的对象关系映射器。

1. 在 **WebApiConfig.cs** 的顶部添加以下代码：

        using AutoMapper;
        using ShoppingService.DataObjects;

2. 若要定义映射，请将以下代码添加到 **WebApiConfig** 类的 **Register** 方法中。

        Mapper.Initialize(cfg =>
        {
            cfg.CreateMap<MobileOrder, Order>();
            cfg.CreateMap<MobileCustomer, Customer>();
            cfg.CreateMap<Order, MobileOrder>()
                .ForMember(dst => dst.MobileCustomerId, map => map.MapFrom(x => x.Customer.Id))
                .ForMember(dst => dst.MobileCustomerName, map => map.MapFrom(x => x.Customer.Name));
            cfg.CreateMap<Customer, MobileCustomer>();

        });

AutoMapper 此时会将对象互相映射。将匹配所有具有相应名称的属性，例如，**MobileOrder.CustomerId** 将会自动映射到 **Order.CustomerId**。可按如下所示配置自定义映射，其中，我们将 **MobileCustomerName** 属性映射到 **Customer** 关系属性的 **Name** 属性。

<a name="DomainManager"></a>
## 实现特定于域的逻辑

下一步是实现 [**MappedEntityDomainManager**](http://msdn.microsoft.com/zh-cn/library/dn643300.aspx)，它用作映射的数据存储与要从客户端提供 HTTP 流量的控制器之间的抽象层。我们可以在下一个部分中完全以 DTO 的形式编写控制器，而在此处添加的 **MappedEntityDomainManager** 将会处理与原始数据存储之间的通信，同时让我们有地方可以实现其任何特定逻辑。

1. 将 **MobileCustomerDomainManager.cs** 添加到项目的 **Models** 文件夹。粘贴以下实现：

        using AutoMapper;
        using Microsoft.WindowsAzure.Mobile.Service;
        using ShoppingService.DataObjects;
        using System.Data.Entity;
        using System.Linq;
        using System.Net.Http;
        using System.Threading.Tasks;
        using System.Web.Http;
        using System.Web.Http.OData;

        namespace ShoppingService.Models
        {
            public class MobileCustomerDomainManager : MappedEntityDomainManager<MobileCustomer, Customer>
            {
                private ExistingContext context;

                public MobileCustomerDomainManager(ExistingContext context, HttpRequestMessage request, ApiServices services)
                    : base(context, request, services)
                {
                    Request = request;
                    this.context = context;
                }

                public static int GetKey(string mobileCustomerId, DbSet<Customer> customers, HttpRequestMessage request)
                {
                    int customerId = customers
                       .Where(c => c.Id == mobileCustomerId)
                       .Select(c => c.CustomerId)
                       .FirstOrDefault();

                    if (customerId == 0)
                    {
                        throw new HttpResponseException(request.CreateNotFoundResponse());
                    }
                    return customerId;
                }

                protected override T GetKey<T>(string mobileCustomerId)
                {
                    return (T)(object)GetKey(mobileCustomerId, this.context.Customers, this.Request);
                }
                
                public override SingleResult<MobileCustomer> Lookup(string mobileCustomerId)
                {
                    int customerId = GetKey<int>(mobileCustomerId);
                    return LookupEntity(c => c.CustomerId == customerId);
                }

                public override async Task<MobileCustomer> InsertAsync(MobileCustomer mobileCustomer)
                {
                    return await base.InsertAsync(mobileCustomer);
                }

                public override async Task<MobileCustomer> UpdateAsync(string mobileCustomerId, Delta<MobileCustomer> patch)
                {
                    int customerId = GetKey<int>(mobileCustomerId);

                    Customer existingCustomer = await this.Context.Set<Customer>().FindAsync(customerId);
                    if (existingCustomer == null)
                    {
                        throw new HttpResponseException(this.Request.CreateNotFoundResponse());
                    }

                    MobileCustomer existingCustomerDTO = Mapper.Map<Customer, MobileCustomer>(existingCustomer);
                    patch.Patch(existingCustomerDTO);
                    Mapper.Map<MobileCustomer, Customer>(existingCustomerDTO, existingCustomer);

                    await this.SubmitChangesAsync();

                    MobileCustomer updatedCustomerDTO = Mapper.Map<Customer, MobileCustomer>(existingCustomer);

                    return updatedCustomerDTO;
                }

                public override async Task<MobileCustomer> ReplaceAsync(string mobileCustomerId, MobileCustomer mobileCustomer)
                {
                    return await base.ReplaceAsync(mobileCustomerId, mobileCustomer);
                }

                public override async Task<bool> DeleteAsync(string mobileCustomerId)
                {
                    int customerId = GetKey<int>(mobileCustomerId);
                    return await DeleteItemAsync(customerId);
                }
            }
        }

    此类的重要部分是 **GetKey** 方法，将在该方法中指示如何在原始数据模型中查找对象的 ID 属性。

2. 将 **MobileOrderDomainManager.cs** 添加到项目的 **Models** 文件夹：

        using AutoMapper;
        using Microsoft.WindowsAzure.Mobile.Service;
        using ShoppingService.DataObjects;
        using System.Linq;
        using System.Net.Http;
        using System.Threading.Tasks;
        using System.Web.Http;
        using System.Web.Http.OData;

        namespace ShoppingService.Models
        {
            public class MobileOrderDomainManager : MappedEntityDomainManager<MobileOrder, Order>
            {
                private ExistingContext context;

                public MobileOrderDomainManager(ExistingContext context, HttpRequestMessage request, ApiServices services)
                    : base(context, request, services)
                {
                    Request = request;
                    this.context = context;
                }

                protected override T GetKey<T>(string mobileOrderId)
                {
                    int orderId = this.context.Orders
                        .Where(o => o.Id == mobileOrderId)
                        .Select(o => o.OrderId)
                        .FirstOrDefault();

                    if (orderId == 0)
                    {
                        throw new HttpResponseException(this.Request.CreateNotFoundResponse());
                    }
                    return (T)(object)orderId;
                }

                public override SingleResult<MobileOrder> Lookup(string mobileOrderId)
                {
                    int orderId = GetKey<int>(mobileOrderId);
                    return LookupEntity(o => o.OrderId == orderId);
                }

                private async Task<Customer> VerifyMobileCustomer(string mobileCustomerId, string mobileCustomerName)
                {
                    int customerId = MobileCustomerDomainManager.GetKey(mobileCustomerId, this.context.Customers, this.Request);
                    Customer customer = await this.context.Customers.FindAsync(customerId);
                    if (customer == null)
                    {
                        throw new HttpResponseException(Request.CreateBadRequestResponse("Customer with name '{0}' was not found", mobileCustomerName));
                    }
                    return customer;
                }

                public override async Task<MobileOrder> InsertAsync(MobileOrder mobileOrder)
                {
                    Customer customer = await VerifyMobileCustomer(mobileOrder.MobileCustomerId, mobileOrder.MobileCustomerName);
                    mobileOrder.CustomerId = customer.CustomerId;
                    return await base.InsertAsync(mobileOrder);
                }

                public override async Task<MobileOrder> UpdateAsync(string mobileOrderId, Delta<MobileOrder> patch)
                {
                    Customer customer = await VerifyMobileCustomer(patch.GetEntity().MobileCustomerId, patch.GetEntity().MobileCustomerName);

                    int orderId = GetKey<int>(mobileOrderId);

                    Order existingOrder = await this.Context.Set<Order>().FindAsync(orderId);
                    if (existingOrder == null)
                    {
                        throw new HttpResponseException(this.Request.CreateNotFoundResponse());
                    }

                    MobileOrder existingOrderDTO = Mapper.Map<Order, MobileOrder>(existingOrder);
                    patch.Patch(existingOrderDTO);
                    Mapper.Map<MobileOrder, Order>(existingOrderDTO, existingOrder);

                    // This is required to map the right Id for the customer
                    existingOrder.CustomerId = customer.CustomerId;

                    await this.SubmitChangesAsync();

                    MobileOrder updatedOrderDTO = Mapper.Map<Order, MobileOrder>(existingOrder);

                    return updatedOrderDTO;
                }

                public override async Task<MobileOrder> ReplaceAsync(string mobileOrderId, MobileOrder mobileOrder)
                {
                    await VerifyMobileCustomer(mobileOrder.MobileCustomerId, mobileOrder.MobileCustomerName);

                    return await base.ReplaceAsync(mobileOrderId, mobileOrder);
                }

                public override Task<bool> DeleteAsync(string mobileOrderId)
                {
                    int orderId = GetKey<int>(mobileOrderId);
                    return DeleteItemAsync(orderId);
                }
            }
        }

    在此情况下，**InsertAsync** 和 **UpdateAsync** 方法是很值得了解的；我们会在其中强制执行每个 **Order** 都需要有一个有效相关 **Customer** 的关系。在 **InsertAsync** 中，你会看到我们填充 **MobileOrder.CustomerId** 属性，此属性映射到 **Order.CustomerId** 属性。可以通过查找具有匹配 **MobileOrder.MobileCustomerId** 的 **Customer** 来获取该值。这是因为，默认情况下，客户端只能识别 **Customer** 的移动服务 ID (**MobileOrder.MobileCustomerId**)，而此 ID 与将外键 (**MobileOrder.CustomerId**) 从 **Order** 设为 **Customer** 所需的实际主键不同。此 ID 仅在服务内部使用，以提高插入操作的速度。

现在我们已可创建控制器，以向客户端公开 DTO。

<a name="Controller"></a>
## 使用 DTO 实现 TableController

1. 在 **Controllers** 文件夹中添加文件 **MobileCustomerController.cs**：

        using Microsoft.WindowsAzure.Mobile.Service;
        using Microsoft.WindowsAzure.Mobile.Service.Security;
        using ShoppingService.DataObjects;
        using ShoppingService.Models;
        using System.Linq;
        using System.Threading.Tasks;
        using System.Web.Http;
        using System.Web.Http.Controllers;
        using System.Web.Http.OData;

        namespace ShoppingService.Controllers
        {
            public class MobileCustomerController : TableController<MobileCustomer>
            {
                protected override void Initialize(HttpControllerContext controllerContext)
                {
                    base.Initialize(controllerContext);
                    var context = new ExistingContext();
                    DomainManager = new MobileCustomerDomainManager(context, Request, Services);
                }

                public IQueryable<MobileCustomer> GetAllMobileCustomers()
                {
                    return Query();
                }

                public SingleResult<MobileCustomer> GetMobileCustomer(string id)
                {
                    return Lookup(id);
                }

                [AuthorizeLevel(AuthorizationLevel.Admin)]
                protected override Task<MobileCustomer> PatchAsync(string id, Delta<MobileCustomer> patch)
                {
                    return base.UpdateAsync(id, patch);
                }

                [AuthorizeLevel(AuthorizationLevel.Admin)]
                protected override Task<MobileCustomer> PostAsync(MobileCustomer item)
                {
                    return base.InsertAsync(item);
                }

                [AuthorizeLevel(AuthorizationLevel.Admin)]
                protected override Task DeleteAsync(string id)
                {
                    return base.DeleteAsync(id);
                }
            }
        }

    你将会注意到，其中已使用 AuthorizeLevel 属性来限制对控制器上的插入/更新/删除操作的公共访问。对于这种情况，客户列表是只读的，但我们将允许创建新订单，并将其与现有客户关联。

2. 在 **Controllers** 文件夹中添加文件 **MobileOrderController.cs**：

        using Microsoft.WindowsAzure.Mobile.Service;
        using ShoppingService.DataObjects;
        using ShoppingService.Models;
        using System.Linq;
        using System.Threading.Tasks;
        using System.Web.Http;
        using System.Web.Http.Controllers;
        using System.Web.Http.Description;
        using System.Web.Http.OData;

        namespace ShoppingService.Controllers
        {
            public class MobileOrderController : TableController<MobileOrder>
            {
                protected override void Initialize(HttpControllerContext controllerContext)
                {
                    base.Initialize(controllerContext);
                    ExistingContext context = new ExistingContext();
                    DomainManager = new MobileOrderDomainManager(context, Request, Services);
                }

                public IQueryable<MobileOrder> GetAllMobileOrders()
                {
                    return Query();
                }

                public SingleResult<MobileOrder> GetMobileOrder(string id)
                {
                    return Lookup(id);
                }

                public Task<MobileOrder> PatchMobileOrder(string id, Delta<MobileOrder> patch)
                {
                    return UpdateAsync(id, patch);
                }

                [ResponseType(typeof(MobileOrder))]
                public async Task<IHttpActionResult> PostMobileOrder(MobileOrder item)
                {
                    MobileOrder current = await InsertAsync(item);
                    return CreatedAtRoute("Tables", new { id = current.Id }, current);
                }

                public Task DeleteMobileOrder(string id)
                {
                    return DeleteAsync(id);
                }
            }
        }

3. 现在，你可以运行服务。按 **F5**，然后使用内置于帮助页中的测试客户端来修改数据。

请注意，这两个控制器实现独占使用 **MobileCustomer** 和 **MobileOrder**，且不区分基础模型。这些 DTO 已序列化为 JSON，并可用来与所有平台上的移动服务客户端 SDK 交换数据。例如，如果生成 Windows 应用商店应用程序，则相应的客户端类型将如下所示。该类型将与其他客户端平台上的类型相似。

    using Microsoft.WindowsAzure.MobileServices;
    using System;

    namespace ShoppingClient
    {
        public class MobileCustomer
        {
            public string Id { get; set; }

            public string Name { get; set; }

            [CreatedAt]
            public DateTimeOffset? CreatedAt { get; set; }

            [UpdatedAt]
            public DateTimeOffset? UpdatedAt { get; set; }

            public bool Deleted { get; set; }
            
            [Version]
            public string Version { get; set; }

        }

    }

接下来，你可以生成客户端应用程序以访问服务。

<!---HONumber=Mooncake_0118_2016-->