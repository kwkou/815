<properties linkid="manage-services-getting-started-with-sqldbs" urlDisplayName="How to create & provision" pageTitle="SQL Database 入门 - Azure" metaKeywords="" description="开始在 Azure 中创建和管理 SQL Database。" metaCanonical="" services="sql-database" documentationCenter="" title="Getting Started with Azure SQL Database" authors="louisb"  solutions="" writer="" manager="jeffreyg" editor="tysonn"  />




#Microsoft Azure SQL Database 入门

在本教程中，您将了解有关使用 Azure 管理门户执行 Microsoft Azure SQL Database 管理任务的基础知识。如果您对数据库管理不熟悉，则可以通过这些课程在大约 30 分钟的时间内学习一些基本技能。 

本教程假定您之前未使用过 SQL Server 或 Azure SQL Database。完成本教程后，您将在 Azure 上拥有一个示例数据库，并了解如何使用管理门户执行基本管理任务。

您将在 Azure 平台上创建和设置一个示例数据库，并使用 Excel 查询系统和用户数据。


##目录##

* [步骤 1：创建 Microsoft Azure 帐户](#Subscribe)
* [步骤 2：连接到 Azure 并创建数据库](#Subscribe)
* [步骤 3：配置防火墙](#ConfigFirewall)
* [步骤 4：使用 Transact-SQL 脚本添加数据和架构](#AddData)
* [步骤 5：创建架构](#createschema)
* [步骤 6：插入数据](#insertData)
* [步骤 7：在 SQL Database 管理门户中查询示例和系统数据](#QueryDBSysData)
* [步骤 8：创建数据库登录名并分配权限](#DBLogin)
* [步骤 9：从应用程序连接](#ClientConnection)


<h2 id="Subscribe">步骤 1：创建 Microsoft Azure 帐户</h2>

1. 打开 Web 浏览器并浏览到 [http://www.windowsazure.cn](http://www.windowsazure.cn)。
若要开始使用免费帐户，请单击右上角的"免费试用"并执行相应步骤。

2. 现已创建您的帐户。一切准备就绪，即可开始。


<h2 id="Connect">步骤 2：连接到 Azure 并创建数据库</h2>


1. 登录到[管理门户](http://manage.windowsazure.cn)。您应当会看到如下所示的导航窗格。

	![Navigation pane][Image1]

2. 单击页面底部的"新建"****。单击"新建"****后，屏幕上将会出现一个显示可创建内容的列表。

3. 单击"SQL Database"****，然后单击"自定义创建"****。 

	![Navigation pane][Image2]

通过选择此选项，您可以以管理员身份同时创建新的服务器和 SQL Database。作为系统管理员，您可以执行更多任务，包括连接到 SQL Database 管理门户，稍后您将在本教程中执行该操作。  

4.  单击"自定义创建"****后，将显示"数据库设置"页。在此页面中，需要提供在服务器上创建空 SQL Database 的基本信息。将在后面的步骤中添加表和数据。 

    如下所示填写"数据库设置"页：

	![Navigation pane][Image3]

* 输入 **School** 作为数据库名称。 

* 使用版本、最大大小和排序规则的默认设置。 

* 选择"新建 SQL Database 服务器"****。选择新服务器时会另外添加一页，可在该页上设置管理员帐户和区域。 

* 完成后，单击箭头转到下一页。


7. 如下所示填写"服务器设置"： 

	![Navigation pane][Image4]

* 输入一个没有空格的单词作为管理员名称。SQL Database 在加密连接中使用 SQL 身份验证来验证用户身份。将使用您提供的名称创建一个具有管理员权限的新 SQL Server 身份验证登录名。管理员名称既不能是 Windows 用户，也不能是 Live ID 用户名。SQL Database 不支持 Windows 身份验证。

* 提供由大小写值以及数字或符号共同组成的 8 个以上字符的强密码。使用帮助气泡以了解有关密码复杂性的详细信息。

* 选择区域。区域将确定服务器的地理位置。区域不能随意切换，因此要选择一个对此服务器有效的区域。选择一个最靠近您的位置。将 Azure 应用程序和数据库放置在同一区域可以降低出口带宽成本以及减少数据延迟情况。

* 确保"允许 Azure 服务访问此服务器"****复选框处于选中状态，以便您能够使用 SQL Database 管理门户、Office 365 中的 Excel 或 Azure SQL 报告连接到此数据库。

* 完成后，请单击页面底部的复选标记。

请注意，您没有指定服务器名称。因为 SQL Database 服务器必须可供全球访问，所以 SQL Database 会在创建服务器后配置适当的 DNS 条目。生成的名称可确保与其他 DNS 条目没有名称冲突。您不能更改 SQL Database 服务器的名称。

若要查看刚创建的用于托管 **School** 数据库的服务器名称，请在左侧导航窗格中单击"SQL Database"****，然后在"SQL Database"****列表视图中单击 **School** 数据库。在"快速启动"****页上，向下滚动该页面以查看该服务器名称。

在下一步中，您将配置防火墙以便允许您计算机上运行的应用程序通过建立连接来访问 SQL Database 服务器上的数据库。



<h2 id="ConfigFirewall">步骤 3：配置防火墙</h2>

若要配置防火墙以便允许连接通过，您需要在服务器页上输入信息。

**请注意：**SQL Database 服务仅适用于 TDS 协议使用的 TCP 端口 1433，因此，请确保您的网络和本地计算机上的防火墙允许端口 1433 上的传出 TCP 通信。有关详细信息，请参阅 [SQL Database 防火墙](http://social.technet.microsoft.com/wiki/contents/articles/2677.sql-azure-firewall-zh-cn.aspx)。


1. 在左侧导航窗格中，单击"SQL Database"****。

2. 单击页面顶部的"服务器"****。接下来，单击您刚创建的服务器以打开服务器页。

3. 在服务器页上，单击"配置"****以打开"允许的 IP 地址"****设置，然后单击"添加到允许的 IP 地址"****链接。这将创建一个新的防火墙规则，以允许来自您的设备正在侦听的路由器或代理服务器的连接请求。

4. 通过指定规则名称以及 IP 范围的起始值和结束值可创建其他防火墙规则。

5. 若要实现此服务器与其他 Azure 服务之间的交互，请单击"是"****以显示"Microsoft Azure 服务"****选项。 

7. 若要保存所做的更改，请单击页面底部的"保存"****。

6. 保存规则后，页面将类似于以下屏幕快照。

	![Navigation pane][Image7]

现在，您已在 Azure 上拥有了 SQL Database 服务器、允许访问该服务器的防火墙规则、数据库对象以及管理员登录名。但是，您仍然没有可查询的工作数据库。为此，您的数据库必须具有架构和实际数据。

由于本教程中仅使用已有的工具，因此您将使用 SQL Database 管理门户中的查询窗口来运行 Transact-SQL 脚本，以便生成预定义的数据库。

随着您的技能的提升，您将希望了解创建数据库的其他方法，包括编程方法或使用 SQL Server Data Tools 中的设计图面。如果您已具有在本地服务器上运行的现有 SQL Server 数据库，则可以轻松地将该数据库迁移到您刚设置的 Azure 服务器上。使用本教程末尾的链接可了解操作方式。 



<h2 id="AddData">步骤 4：使用 Transact-SQL 脚本添加数据和架构</h2>

在本步骤中，您需要运行两个脚本。第一个脚本创建一个定义表、列和关系的架构。第二个脚本添加数据。每个步骤在单独的连接中独立执行。如果您之前已在 SQL Server 中构建数据库，那么您将在 SQL Database 中看到的差异之一是 CREATE 和 INSERT 命令必须在单独的批处理文件中运行。SQL Database 实施此要求是为了最大程度地减少数据在传送过程中受到的攻击。 

**请注意：**架构和数据值摘自此 [MSDN 文章](http://msdn.microsoft.com/zh-cn/library/azure/ee621790.aspx "MSDN article")且已经过修改，可用于 SQL Database。

1. 转到主页。在[管理门户](http://manage.windowsazure.cn)中，**School** 数据库将出现在主页上的项列表中。

	![Navigation pane][Image8]

2. 单击 **School** 以将其选中，然后单击页面底部的"管理"****。这将打开 SQL Database 管理门户。此门户与 Azure 管理门户不同。您将使用此门户运行 Transact-SQL 命令和查询。

3. 输入用于登录到 **School** 数据库的管理员登录名和密码。这是您在创建服务器时所指定的管理员登录名。

4. 在 SQL Database 管理门户的功能区中，单击"新建查询"****。将在工作区中打开一个空查询窗口。在下一步中，您将使用此窗口来复制用于将结构和数据添加到空数据库的一系列预定义脚本。



<h2 id="createschema">步骤 5：创建架构</h2>

在本步骤中，您将使用以下脚本来创建架构。该脚本首先会检查相同名称的现有表，以确保不会出现名称冲突，并使用 [CREATE TABLE](http://msdn.microsoft.com/zh-cn/library/azure/ee336258.aspx) 语句创建表。然后，此脚本使用 [ALTER TABLE](http://msdn.microsoft.com/zh-cn/library/azure/ee336286.aspx) 语句指定主密钥和表关系。

复制该脚本并将其粘贴到查询窗口中。单击窗口顶部的"运行"****以执行该脚本。

<div style="width:auto; height:600px; overflow:auto"><pre>
	-- Create the Department table.
	IF NOT EXISTS (SELECT * FROM sys.objects 
		WHERE object_id = OBJECT_ID(N'[dbo].[Department]') 
		AND type in (N'U'))
   	BEGIN
  	CREATE TABLE [dbo].[Department](
		[DepartmentID] [int] NOT NULL,
		[Name] [nvarchar](50) NOT NULL,
		[Budget] [money] NOT NULL,
		[StartDate] [datetime] NOT NULL,
		[Administrator] [int] NULL,
     CONSTRAINT [PK_Department] PRIMARY KEY CLUSTERED 
    (
	[DepartmentID] ASC
    )WITH (IGNORE_DUP_KEY = OFF)
    )
    END;
	GO

	-- Create the Person table.
	IF NOT EXISTS (SELECT * FROM sys.objects 
		WHERE object_id = OBJECT_ID(N'[dbo].[Person]') 
		AND type in (N'U'))
	BEGIN
	CREATE TABLE [dbo].[Person](
		[PersonID] [int] IDENTITY(1,1) NOT NULL,
		[LastName] [nvarchar](50) NOT NULL,
		[FirstName] [nvarchar](50) NOT NULL,
		[HireDate] [datetime] NULL,
		[EnrollmentDate] [datetime] NULL,
 	 CONSTRAINT [PK_School.Student] PRIMARY KEY CLUSTERED 	
	(
	[PersonID] ASC
	)WITH (IGNORE_DUP_KEY = OFF)
	) 
	END;
	GO

	-- Create the OnsiteCourse table.
	IF NOT EXISTS (SELECT * FROM sys.objects 
		WHERE object_id = OBJECT_ID(N'[dbo].[OnsiteCourse]') 
		AND type in (N'U'))
	BEGIN
	CREATE TABLE [dbo].[OnsiteCourse](
		[CourseID] [int] NOT NULL,
		[Location] [nvarchar](50) NOT NULL,
		[Days] [nvarchar](50) NOT NULL,
		[Time] [smalldatetime] NOT NULL,
 	 CONSTRAINT [PK_OnsiteCourse] PRIMARY KEY CLUSTERED 
	(
		[CourseID] ASC
	)WITH (IGNORE_DUP_KEY = OFF)
	) 
	END;
	GO

	-- Create the OnlineCourse table.
	IF NOT EXISTS (SELECT * FROM sys.objects 
		WHERE object_id = OBJECT_ID(N'[dbo].[OnlineCourse]') 
		AND type in (N'U'))
	BEGIN
	CREATE TABLE [dbo].[OnlineCourse](
		[CourseID] [int] NOT NULL,
		[URL] [nvarchar](100) NOT NULL,
 	 CONSTRAINT [PK_OnlineCourse] PRIMARY KEY CLUSTERED 
	(
		[CourseID] ASC
	)WITH (IGNORE_DUP_KEY = OFF)
	) 
	END;
	GO

	--Create the StudentGrade table.
	IF NOT EXISTS (SELECT * FROM sys.objects 
		WHERE object_id = OBJECT_ID(N'[dbo].[StudentGrade]') 
		AND type in (N'U'))
	BEGIN
	CREATE TABLE [dbo].[StudentGrade](
		[EnrollmentID] [int] IDENTITY(1,1) NOT NULL,
		[CourseID] [int] NOT NULL,
		[StudentID] [int] NOT NULL,
		[Grade] [decimal](3, 2) NULL,
	 CONSTRAINT [PK_StudentGrade] PRIMARY KEY CLUSTERED 
	(
		[EnrollmentID] ASC
	)WITH (IGNORE_DUP_KEY = OFF)
	) 
	END;
	GO

	-- Create the CourseInstructor table.
	IF NOT EXISTS (SELECT * FROM sys.objects 
		WHERE object_id = OBJECT_ID(N'[dbo].[CourseInstructor]') 
		AND type in (N'U'))
	BEGIN
	CREATE TABLE [dbo].[CourseInstructor](
		[CourseID] [int] NOT NULL,
		[PersonID] [int] NOT NULL,
	 CONSTRAINT [PK_CourseInstructor] PRIMARY KEY CLUSTERED 
	(
		[CourseID] ASC,
		[PersonID] ASC
	)WITH (IGNORE_DUP_KEY = OFF)
	) 
	END;
	GO

	-- Create the Course table.
	IF NOT EXISTS (SELECT * FROM sys.objects 
		WHERE object_id = OBJECT_ID(N'[dbo].[Course]') 
		AND type in (N'U'))
	BEGIN
	CREATE TABLE [dbo].[Course](
		[CourseID] [int] NOT NULL,
		[Title] [nvarchar](100) NOT NULL,
		[Credits] [int] NOT NULL,
		[DepartmentID] [int] NOT NULL,
 	 CONSTRAINT [PK_School.Course] PRIMARY KEY CLUSTERED 
	(
		[CourseID] ASC
	)WITH (IGNORE_DUP_KEY = OFF)
	)
	END;
	GO

	-- Create the OfficeAssignment table.
	IF NOT EXISTS (SELECT * FROM sys.objects 
		WHERE object_id = OBJECT_ID(N'[dbo].[OfficeAssignment]')
		AND type in (N'U'))
	BEGIN
	CREATE TABLE [dbo].[OfficeAssignment](
		[InstructorID] [int] NOT NULL,
		[Location] [nvarchar](50) NOT NULL,
		[Timestamp] [timestamp] NOT NULL,
 	 CONSTRAINT [PK_OfficeAssignment] PRIMARY KEY CLUSTERED 
	(
		[InstructorID] ASC
	)WITH (IGNORE_DUP_KEY = OFF)
	)
	END;
	GO

	-- Define the relationship between OnsiteCourse and Course.
	IF NOT EXISTS (SELECT * FROM sys.foreign_keys 
       WHERE object_id = OBJECT_ID(N'[dbo].[FK_OnsiteCourse_Course]')
       AND parent_object_id = OBJECT_ID(N'[dbo].[OnsiteCourse]'))
	ALTER TABLE [dbo].[OnsiteCourse]  WITH CHECK ADD  
       CONSTRAINT [FK_OnsiteCourse_Course] FOREIGN KEY([CourseID])
	REFERENCES [dbo].[Course] ([CourseID]);
	GO
	ALTER TABLE [dbo].[OnsiteCourse] CHECK 
       CONSTRAINT [FK_OnsiteCourse_Course];
	GO

	-- Define the relationship between OnlineCourse and Course.
	IF NOT EXISTS (SELECT * FROM sys.foreign_keys 
       WHERE object_id = OBJECT_ID(N'[dbo].[FK_OnlineCourse_Course]')
       AND parent_object_id = OBJECT_ID(N'[dbo].[OnlineCourse]'))
	ALTER TABLE [dbo].[OnlineCourse]  WITH CHECK ADD  
       CONSTRAINT [FK_OnlineCourse_Course] FOREIGN KEY([CourseID])
	REFERENCES [dbo].[Course] ([CourseID]);
	GO
	ALTER TABLE [dbo].[OnlineCourse] CHECK 
       CONSTRAINT [FK_OnlineCourse_Course];
	GO
	-- Define the relationship between StudentGrade and Course.
	IF NOT EXISTS (SELECT * FROM sys.foreign_keys 
       WHERE object_id = OBJECT_ID(N'[dbo].[FK_StudentGrade_Course]')
       AND parent_object_id = OBJECT_ID(N'[dbo].[StudentGrade]'))
	ALTER TABLE [dbo].[StudentGrade]  WITH CHECK ADD  
       CONSTRAINT [FK_StudentGrade_Course] FOREIGN KEY([CourseID])
	REFERENCES [dbo].[Course] ([CourseID]);
	GO
	ALTER TABLE [dbo].[StudentGrade] CHECK 
       CONSTRAINT [FK_StudentGrade_Course];
	GO

	--Define the relationship between StudentGrade and Student.
	IF NOT EXISTS (SELECT * FROM sys.foreign_keys 
       WHERE object_id = OBJECT_ID(N'[dbo].[FK_StudentGrade_Student]')
       AND parent_object_id = OBJECT_ID(N'[dbo].[StudentGrade]'))	
	ALTER TABLE [dbo].[StudentGrade]  WITH CHECK ADD  
       CONSTRAINT [FK_StudentGrade_Student] FOREIGN KEY([StudentID])
	REFERENCES [dbo].[Person] ([PersonID]);
	GO
	ALTER TABLE [dbo].[StudentGrade] CHECK 
       CONSTRAINT [FK_StudentGrade_Student];
	GO

	-- Define the relationship between CourseInstructor and Course.
	IF NOT EXISTS (SELECT * FROM sys.foreign_keys 
  	 WHERE object_id = OBJECT_ID(N'[dbo].[FK_CourseInstructor_Course]')
  	 AND parent_object_id = OBJECT_ID(N'[dbo].[CourseInstructor]'))
	ALTER TABLE [dbo].[CourseInstructor]  WITH CHECK ADD  
  	 CONSTRAINT [FK_CourseInstructor_Course] FOREIGN KEY([CourseID])
	REFERENCES [dbo].[Course] ([CourseID]);
	GO
	ALTER TABLE [dbo].[CourseInstructor] CHECK 
 	  CONSTRAINT [FK_CourseInstructor_Course];
	GO

	-- Define the relationship between CourseInstructor and Person.
	IF NOT EXISTS (SELECT * FROM sys.foreign_keys 
 	  WHERE object_id = OBJECT_ID(N'[dbo].[FK_CourseInstructor_Person]')
	   AND parent_object_id = OBJECT_ID(N'[dbo].[CourseInstructor]'))
	ALTER TABLE [dbo].[CourseInstructor]  WITH CHECK ADD  
 	  CONSTRAINT [FK_CourseInstructor_Person] FOREIGN KEY([PersonID])
	REFERENCES [dbo].[Person] ([PersonID]);
	GO
	ALTER TABLE [dbo].[CourseInstructor] CHECK 
  	 CONSTRAINT [FK_CourseInstructor_Person];
	GO

	-- Define the relationship between Course and Department.
	IF NOT EXISTS (SELECT * FROM sys.foreign_keys 
       WHERE object_id = OBJECT_ID(N'[dbo].[FK_Course_Department]')
       AND parent_object_id = OBJECT_ID(N'[dbo].[Course]'))
	ALTER TABLE [dbo].[Course]  WITH CHECK ADD  
       CONSTRAINT [FK_Course_Department] FOREIGN KEY([DepartmentID])
	REFERENCES [dbo].[Department] ([DepartmentID]);
	GO
	ALTER TABLE [dbo].[Course] CHECK CONSTRAINT [FK_Course_Department];
	GO

	--Define the relationship between OfficeAssignment and Person.
	IF NOT EXISTS (SELECT * FROM sys.foreign_keys 
	  WHERE object_id = OBJECT_ID(N'[dbo].[FK_OfficeAssignment_Person]')
 	  AND parent_object_id = OBJECT_ID(N'[dbo].[OfficeAssignment]'))
	ALTER TABLE [dbo].[OfficeAssignment]  WITH CHECK ADD  
 	  CONSTRAINT [FK_OfficeAssignment_Person] FOREIGN KEY([InstructorID])
	REFERENCES [dbo].[Person] ([PersonID]);
	GO
	ALTER TABLE [dbo].[OfficeAssignment] CHECK 
   	 CONSTRAINT [FK_OfficeAssignment_Person];
	GO
</pre></div>



<h2 id="insertData">步骤 6：插入数据</h2>

打开一个新查询窗口，然后粘贴以下脚本。运行该脚本以插入数据。此脚本使用 [INSERT](http://msdn.microsoft.com/zh-cn/library/azure/ee336284.aspx) 语句将值添加到各列。

<div style="width:auto; height:600px; overflow:auto"><pre>
	-- Insert data into the Person table.
	SET IDENTITY_INSERT dbo.Person ON;
	GO
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (1, 'Abercrombie', 'Kim', '1995-03-11', null);
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (2, 'Barzdukas', 'Gytis', null, '2005-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (3, 'Justice', 'Peggy', null, '2001-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (4, 'Fakhouri', 'Fadi', '2002-08-06', null);
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (5, 'Harui', 'Roger', '1998-07-01', null);
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (6, 'Li', 'Yan', null, '2002-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (7, 'Norman', 'Laura', null, '2003-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (8, 'Olivotto', 'Nino', null, '2005-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (9, 'Tang', 'Wayne', null, '2005-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (10, 'Alonso', 'Meredith', null, '2002-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (11, 'Lopez', 'Sophia', null, '2004-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (12, 'Browning', 'Meredith', null, '2000-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (13, 'Anand', 'Arturo', null, '2003-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (14, 'Walker', 'Alexandra', null, '2000-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (15, 'Powell', 'Carson', null, '2004-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (16, 'Jai', 'Damien', null, '2001-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (17, 'Carlson', 'Robyn', null, '2005-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (18, 'Zheng', 'Roger', '2004-02-12', null);
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (19, 'Bryant', 'Carson', null, '2001-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (20, 'Suarez', 'Robyn', null, '2004-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (21, 'Holt', 'Roger', null, '2004-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (22, 'Alexander', 'Carson', null, '2005-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (23, 'Morgan', 'Isaiah', null, '2001-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (24, 'Martin', 'Randall', null, '2005-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (25, 'Kapoor', 'Candace', '2001-01-15', null);
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (26, 'Rogers', 'Cody', null, '2002-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (27, 'Serrano', 'Stacy', '1999-06-01', null);
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (28, 'White', 'Anthony', null, '2001-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (29, 'Griffin', 'Rachel', null, '2004-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (30, 'Shan', 'Alicia', null, '2003-09-01');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (31, 'Stewart', 'Jasmine', '1997-10-12', null);
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (32, 'Xu', 'Kristen', '2001-7-23', null);
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (33, 'Gao', 'Erica', null, '2003-01-30');
	INSERT INTO dbo.Person (PersonID, LastName, FirstName, HireDate, EnrollmentDate)
	VALUES (34, 'Van Houten', 'Roger', '2000-12-07', null);
	GO
	SET IDENTITY_INSERT dbo.Person OFF;
	GO
	
</pre></div>


<h2 id="QueryDBSysData">步骤 7：在 SQL Database 管理门户中查询示例和系统数据</h2>

若要检查您的工作，请运行可返回您刚输入的数据的查询。您也可以运行内置存储过程和数据管理视图，这些视图可提供有关 SQL Database 服务器上运行的数据库的信息。

<h4 id="QueryDB">查询示例数据</h4>

在新查询窗口中，复制并运行以下 Transact-SQL 脚本以检索您刚添加的一些数据。


<div style="width:auto; height:auto; overflow:auto"><pre>
	SELECT * From Person
</pre></div>

您应该会在 person 表中看到一个带有 34 行的结果集，包括 PersonID、LastName、FirstName、HireDate 和 EnrollmentDate。


<h4 id="QuerySys">查询系统数据</h4>

您也可以使用系统视图和内置存储过程来从服务器获取信息。对于本教程，您将尝试几个命令。

运行以下命令可确定服务器中的可用数据库。 

	SELECT * FROM sys.databases  

运行此命令可返回当前连接到服务器的用户列表。

	SELECT user_name(),suser_sname()

运行此存储过程以返回 **School** 数据库中所有对象的列表。

	EXEC SP_help

不要关闭与 **School** 数据库的门户连接。因为几分钟后您需要再次使用该连接。



<h2 id="DBLogin">步骤 8：创建数据库登录名并分配权限</h2>

在 SQL Database 中，您可以使用 Transact-SQL 创建登录名和授予权限。在本课程中，您将使用 Transact-SQL 执行以下三项操作：


1. 创建 SQL Server 身份验证登录名
2. 创建数据库用户，并
3. 通过角色成员身份授予权限。

SQL Server 身份验证登录名用于建立服务器连接。访问 SQL Database 服务器上数据库的所有用户均要通过提供 SQL Server 身份验证登录名和密码来建立连接。 

若要创建登录名，您必须先连接到 **master** 数据库。

<h4 id="CreateLogin">创建 SQL Server 身份验证登录名</h4>

1. 在[管理门户](http://manage.windowsazure.cn)中，选择"SQL Database"****、单击"服务器"****、选择所需服务器，然后单击白色箭头以打开服务器页。

2. 在"快速启动"页上，单击"管理服务器"****以打开一个指向 SQL Database 管理门户的新连接。 

3. 指定要连接的 **master** 数据库，然后使用您的用户名和密码登录。这是您在创建服务器时所指定的管理员登录名。

4. 将在新浏览器窗口中打开 SQL Database 管理门户并且会将您连接到 **master**。

5. 如果您在该页面上看到类似如下错误，请忽略它。单击"新建查询"****以打开一个允许您对 **master** 数据库执行 Transact-SQL 命令的查询窗口。

	![Navigation pane][Image15]

6. 将以下命令复制并粘贴到该查询窗口中。

        CREATE LOGIN SQLDBLogin WITH password='Password1';

7. 运行该命令以创建一个名为"SQLDBLogin"的新 SQL Server 登录名。


<h4 id="CreateDBuser">创建数据库用户并分配权限</h4>

在您创建 SQL Server 身份验证登录名后，下一步是分配与该登录名关联的数据库和权限级别。通过在每个数据库上创建"数据库用户"****可执行此操作。

1. 返回到可连接 **School** 数据库的 SQL Database 管理门户页。如果您关闭了浏览器窗口，请使用上一课程"使用 Transact-SQL 脚本添加数据和架构"中的步骤启动一个到 **School** 数据库的新连接。 

	在 SQL Database 管理门户页上，**School** 数据库名称将显示在左上角。

	![Navigation pane][Image12]

2. 单击"新建查询"****以打开一个新查询窗口并复制以下语句。 

	    CREATE USER SQLDBUser FROM LOGIN SQLDBLogin;

3. 运行该脚本。此脚本将基于该登录名创建一个新数据库用户。

   接下来，您将使用 db_datareader 角色分配权限。分配给此角色的数据库用户可读取数据库中所有用户表中的全部数据。 

4. 打开一个新查询窗口，然后输入并运行下一个语句。此语句将运行内置存储过程，以便将 db_datareader 角色分配给您刚创建的新用户。 

        EXEC sp_addrolemember 'db_datareader', 'SQLDBUser';

现在，您拥有了一个对 **School** 数据库具有只读权限的新 SQL Server 身份验证登录名。使用这些步骤，您可以创建其他 SQL Server 身份验证登录名，以允许对您的数据进行不同级别的访问。


<h2 id="ClientConnection">步骤 9：从应用程序连接</h2>

可以使用 ADO.NET 连接到 Microsoft Azure SQL Database。与本地连接不同，您需要考虑限制或可终止连接或暂时阻止新连接的其他服务故障。这种情况称为暂时性故障。若要管理暂时性故障，您需要实现重试策略。在连接到 Azure SQL Database 时，[暂时性故障处理应用程序块](http://go.microsoft.com/fwlink/?LinkId=519356)（Enterprise Library 6 ¨C（2013 年 4 月）中的一部分）具有标识暂时性故障状况的检测策略。

<h4>示例 C# 控制台应用程序</h4>


	static void Main(string[] args)
    {
        //NOTE: Use appropriate exception handling in a production application.

        //Replace
        //  builder["Server"]: {servername} = Your Azure SQL Database server name
        //  builder["User ID"]: {username}@{servername} = Your Azure SQL Database user name and server name
        //  builder["Password"]: {password} = Your Azure SQL Database password

        System.Data.SqlClient.SqlConnectionStringBuilder builder = new System.Data.SqlClient.SqlConnectionStringBuilder();
        builder["Server"] = "{servername}";
        builder["User ID"] = "{username}@{servername}";
        builder["Password"] = "{password}";

        builder["Database"] = "AdventureWorks2012";
        builder["Trusted_Connection"] = false;
        builder["Integrated Security"] = false;
        builder["Encrypt"] = true;

        //1. Define an Exponential Backoff retry strategy for Azure SQL Database throttling (ExponentialBackoff Class). An exponential back-off strategy will gracefully back off the load on the service.
        int retryCount = 4;
        int minBackoffDelayMilliseconds = 2000;
        int maxBackoffDelayMilliseconds = 8000;
        int deltaBackoffMilliseconds = 2000;

        ExponentialBackoff exponentialBackoffStrategy = 
          new ExponentialBackoff("exponentialBackoffStrategy",
              retryCount,
              TimeSpan.FromMilliseconds(minBackoffDelayMilliseconds), 
              TimeSpan.FromMilliseconds(maxBackoffDelayMilliseconds),
              TimeSpan.FromMilliseconds(deltaBackoffMilliseconds));

        //2. Set a default strategy to Exponential Backoff.
        RetryManager manager = new RetryManager(new List<RetryStrategy>
        {  
            exponentialBackoffStrategy 
        }, "exponentialBackoffStrategy");

        //3. Set a default Retry Manager. A RetryManager provides retry functionality, or if you are using declarative configuration, you can invoke the RetryPolicyFactory.CreateDefault
            RetryManager.SetDefault(manager);

        //4. Define a default SQL Connection retry policy and SQL Command retry policy. A policy provides a retry mechanism for unreliable actions and transient conditions.
        RetryPolicy retryConnectionPolicy = manager.GetDefaultSqlConnectionRetryPolicy();
        RetryPolicy retryCommandPolicy = manager.GetDefaultSqlCommandRetryPolicy();

        //5. Create a function that will retry the connection using a ReliableSqlConnection.
        retryConnectionPolicy.ExecuteAction(() =>
        {
            using (ReliableSqlConnection connection = new ReliableSqlConnection(builder.ConnectionString))
            {
                connection.Open();

                IDbCommand command = connection.CreateCommand();
                command.CommandText = "SELECT Name FROM Production.Product";

                //6. Create a function that will retry the command calling ExecuteCommand() from the ReliableSqlConnection
                retryCommandPolicy.ExecuteAction(() =>
                {
                    using (IDataReader reader = connection.ExecuteCommand<IDataReader>(command))
                    {
                        while (reader.Read())
                        {
                            string name = reader.GetString(0);

                            Console.WriteLine(name);
                        }
                    }
                });                  
            }
        });

        Console.ReadLine();
    }



<h2 id="NextSteps">后续步骤</h2>

现在，您已了解 SQL Database 和管理门户，接下来，您可以尝试 SQL Server 数据库管理员使用的其他一些工具和方法。

若要主动管理新数据库，请考虑安装和使用 SQL Server Management Studio。Management Studio 是用于管理 SQL Server 数据库（包括在 Azure 上运行的数据库）的主要数据库管理工具。使用 Management Studio，您可以保存查询以供将来使用、添加新表和存储过程，以及磨练您在丰富脚本环境（包括语法检查程序、智能感知和模板）中掌握的 Transact-SQL 技能。若要开始尝试，请按照[使用 SQL Server Management Studio 管理 SQL Database](/zh-cn/documentation/articles/sql-database-manage-azure-ssms/) 中的说明操作。

流利使用 Transact-SQL 查询和数据定义语言对数据库管理员而言至关重要。如果您是第一次使用 Transact-SQL，请从[教程：编写 Transact-SQL 语句](http://msdn.microsoft.com/zh-cn/library/ms365303.aspx)以了解一些基本技能。

也可采用其他方法将本地数据库移动到 SQL Database。如果您拥有现有数据库，或者下载了示例数据库进行练习，请尝试以下替代方法：

* [将数据库迁移到 SQL Database](http://msdn.microsoft.com/zh-cn/library/azure/ee730904.aspx)
* [在 SQL Database 中复制数据库](http://msdn.microsoft.com/zh-cn/library/azure/ff951624.aspx)
* [将 SQL Server 数据库部署到 Azure 虚拟机](http://msdn.microsoft.com/zh-cn/library/dn195938(v=sql.120).aspx)



[Image1]: ./media/sql-database-get-started/1NavPaneDBSelected_SQLTut.png
[Image2]: ./media/sql-database-get-started/2MainPageCustomCreateDB_SQLTut.png
[Image3]: ./media/sql-database-get-started/3DatabaseSettings_SQLTut.PNG
[Image4]: ./media/sql-database-get-started/4ServerSettings_SQLTut.PNG
[Image5]: ./media/sql-database-get-started/5DBPortalDatabasesServers_SQLTut.PNG
[Image6]: ./media/sql-database-get-started/6DBConfigFirewall_SQLTut.PNG
[Image7]: ./media/sql-database-get-started/7DBConfigFirewallSAVE_SQLTut.png
[Image8]: ./media/sql-database-get-started/20MainPageHome_SQLTut.PNG
[Image9]: ./media/sql-database-get-started/9dblistschool_SQLTut.PNG
[Image10]: ./media/sql-database-get-started/10dbportalmanagebutton_SQLTut.PNG
[Image11]: ./media/sql-database-get-started/11ManageDatabaseLogin_SQLTut.PNG
[Image12]: ./media/sql-database-get-started/12DBPortalNewQuery_SQLTut.PNG
[Image13]: ./media/sql-database-get-started/13DBQueryResults_SQLTut.PNG
[Image14]: ./media/sql-database-get-started/14DBPortalConnectMaster_SQLTut.PNG
[Image15]: ./media/sql-database-get-started/15DBPortalConnectMasterErr_SQLTut.PNG
[Image16]: ./media/sql-database-get-started/16ExcelConnect_SQLTut.png
[Image17]: ./media/sql-database-get-started/17ExcelSelect_SQLTut.PNG
[Image18]: ./media/sql-database-get-started/18ExcelTable_SQLTut.PNG
[Image19]: ./media/sql-database-get-started/19ExcelImport_SQLTut.png
[Image20]: ./media/sql-database-get-started/11ManageDatabaseLogin_SQLTut.PNG

