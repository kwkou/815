<properties linkid="manage-services-getting-started-with-sqldbs" urlDisplayName="How to create & provision" pageTitle="SQL Database 入门 - Azure" metaKeywords="" description="开始在 Azure 中创建和管理 SQL Database。" metaCanonical="" services="sql-database" documentationCenter="" title="Azure SQL Database 入门" authors=""  solutions="" writer="" manager="" editor=""  />

# Azure SQL Database 入门

在本教程中，您将了解有关使用 Azure 管理门户执行 Azure SQL Database 管理任务的基础知识。如果您对数据库管理不熟悉，则可以通过这些课程在大约 30 分钟的时间内学习一些基本技能。

本教程假定您之前未使用过 SQL Server 或 Azure SQL Database。完成本教程后，您将在 Azure 上拥有一个示例数据库，并了解如何使用管理门户执行基本管理任务。

您将在 Azure 上创建和设置一个示例数据库，并使用 Excel 和其他应用程序查询系统和用户数据。

## 目录

-   [步骤 1：创建 Azure 帐户][步骤 1：创建 Azure 帐户]
-   [步骤 2：连接到 Azure 并创建数据库][步骤 2：连接到 Azure 并创建数据库]
-   [步骤 3：配置防火墙][步骤 3：配置防火墙]
-   [步骤 4：使用 Transact-SQL 脚本添加数据和架构][步骤 4：使用 Transact-SQL 脚本添加数据和架构]
-   [步骤 5：创建架构][步骤 5：创建架构]
-   [步骤 6：插入数据][步骤 6：插入数据]
-   [步骤 7：在 SQL Database 管理门户中查询示例和系统数据][步骤 7：在 SQL Database 管理门户中查询示例和系统数据]
-   [步骤 8：创建数据库登录名并分配权限][步骤 8：创建数据库登录名并分配权限]
-   [步骤 9：从其他应用程序进行连接][步骤 9：从其他应用程序进行连接]
    <!-- * [Step 10: Configure SQL Data Sync](#ConfigureDataSync) -->

## <span id="Subscribe"></span></a>步骤 1：创建 Azure 帐户

1.  打开 Web 浏览器，并浏览到 <http://www.windowsazure.cn>。
    若要开始使用免费帐户，请单击右上角的“免费试用”，然后按照步骤进行操作。

2.  现已创建您的帐户。一切准备就绪，即可开始。

## <span id="Connect"></span></a>步骤 2：连接到 Azure 并创建数据库

1.  登录到[管理门户][管理门户]。您应当会看到如下所示的导航窗格。

    ![图像 1][图像 1]

2.  单击页面底部的“新建”。单击“新建”后，屏幕上将会出现一个显示可创建内容的列表。

3.  单击“SQL Database”，然后单击“自定义创建”。

    ![图像 2][图像 2]

    选择此选项时，您还能够以管理员身份创建一个新服务器。作为系统管理员，您可以执行更多任务，包括连接到 SQL Database 管理门户，本教程的后面将会介绍该任务。

4.  单击“自定义创建”后，将显示“数据库设置”页。在此页面中，您需要提供在服务器上创建空数据库的基本信息。将在后面的步骤中添加表和数据。

    如下所示填写“数据库设置”页：

    ![图像 3][图像 3]

    -   输入 **School** 作为数据库名称。

    -   使用版本、最大大小和排序规则的默认设置。

    -   选择“新建 SQL Database 服务器”。选择新服务器时会另外添加一页，可在该页上设置管理员帐户和区域。

    -   完成后，单击箭头转到下一页。

5.  如下所示填写“服务器设置”：

    ![图像 4][图像 4]

    -   输入一个没有空格的单词作为管理员名称。SQL Database 在加密连接中使用 SQL 身份验证来验证用户身份。将使用您提供的名称创建一个具有管理员权限的新 SQL Server 身份验证登录名。管理员名称既不能是 Windows 用户，也不能是 Windows Live ID。SQL Database 不支持 Windows 身份验证。

    -   提供由大小写值以及数字或符号共同组成的 8 个以上字符的强密码。

    -   选择区域。区域将确定服务器的地理位置。区域不能随意切换，因此要选择一个对此服务器有效的区域。选择一个最靠近您或您用户的位置。将 Azure 应用程序和数据库放置在同一区域可以降低出口带宽成本以及减少数据延迟情况。

    -   确保“允许 Azure 服务访问此服务器”复选框处于选中状态，以便您能够使用 SQL Database 管理门户、Office 365 中的 Excel 或 Azure SQL 报告连接到此数据库。

    -   完成后，请单击页面底部的复选标记。

    请注意，您没有指定服务器名称。因为 SQL Database 服务器必须可供全球访问，所以 SQL Database 会在创建服务器后配置适当的 DNS 条目。生成的名称可确保与其他 DNS 条目没有名称冲突。您不能更改 SQL Database 服务器的名称。

    在下一步中，您将配置防火墙以便允许您计算机上运行的应用程序通过建立连接来访问 SQL Database 服务器上的数据库。

## <span id="ConfigFirewall"></span></a>步骤 3：配置防火墙

若要配置防火墙以便允许连接通过，您需要在服务器页上输入信息。

**注意：**SQL Database 服务仅适用于 TDS 协议使用的 TCP 端口 1433，因此，请确保您的网络和本地计算机上的防火墙允许端口 1433 上的传出 TCP 通信。有关详细信息，请参阅 [SQL Database 防火墙][SQL Database 防火墙]。

1.  在左侧导航窗格中，单击“SQL Database”。

2.  单击页面顶部的“服务器”。接下来，单击您刚才创建的服务器，您会看到一个向右的白色箭头。单击该箭头以打开服务器页。

    ![图像 5][图像 5]

3.  在服务器页上，单击“配置”以打开防火墙配置设置并按以下方式指定规则：

    ![图像 6][图像 6]

    -   复制当前客户端 IP 地址。它是您的路由器或代理服务器侦听的 IP 地址。SQL Database 会检测当前连接所使用的 IP 地址，以便您可以创建一个接受来自该设备的连接请求的防火墙规则。

    -   将 IP 地址粘贴到起始和结束地址范围中。日后，如果您遇到指示该范围太窄的连接错误，则可以编辑此规则来扩大范围。

    -   为防火墙规则输入名称，例如，您的计算机或公司的名称。

    -   单击复选标记保存该规则。

    保存规则后，页面将类似于以下屏幕快照。

    ![图像 7][图像 7]

4.  单击页面底部的“保存”以完成该步骤。如果没有看到“保存”，请刷新浏览器页面。

现在，您已在 Azure 上拥有了 SQL Database 服务器、允许访问该服务器的防火墙规则、数据库对象以及管理员登录名。接下来，您将使用 SQL Database 管理门户中的查询窗口运行 Transact-SQL 脚本，以创建一个预定义数据库。

随着您的技能的提升，您将希望了解创建数据库的其他方法，包括编程方法或使用 SQL Server Data Tools 中的设计器。如果您已具有在本地服务器上运行的现有 SQL Server 数据库，则可以轻松地将该数据库迁移到您刚设置的 Azure 服务器上。使用本教程末尾的链接可了解操作方式。

## <span id="AddData"></span></a>步骤 4：使用 Transact-SQL 脚本添加数据和架构

在本步骤中，您需要运行两个脚本。第一个脚本创建一个定义表、列和关系的架构。第二个脚本添加数据。每个步骤在单独的连接中独立执行。如果您之前已在 SQL Server 中构建数据库，那么您将在 SQL Database 中看到的差异之一是 CREATE 和 INSERT 命令必须在单独的批处理文件中运行。SQL Database 实施此要求是为了最大程度地减少数据在传送过程中受到的攻击。

**注意：**相关架构和数据值摘自此 [MSDN 文章][MSDN 文章]且已经过修改，可用于 SQL Database。

1.  转到主页。在[管理门户][管理门户]中，**School** 数据库将出现在主页上的项列表中。

    ![图像 8][图像 8]

2.  单击 **School**，您可以看到一个向右的白色箭头。单击该箭头以打开数据库页。

    ![图像 9][图像 9]

3.  单击页面底部的“管理”。如果看不到“管理”，请刷新浏览器窗口。这将打开 SQL Database 管理门户。此门户与 Azure 管理门户不同。您将使用此门户运行 Transact-SQL 命令和查询。

    ![图像 10][图像 10]

4.  输入管理员登录名和密码。这是您在创建服务器时所指定的管理员登录名。

    ![图像 11][图像 11]

5.  在 SQL Database 管理门户中单击“新建查询”。将在工作区中打开一个空查询窗口。在下一步中，您将使用此窗口来复制用于将结构和数据添加到空数据库的一系列预定义脚本。

    ![图像 12][图像 12]

## <span id="createschema"></span></a>步骤 5：创建架构

在本步骤中，您将使用以下脚本来创建架构。该脚本首先会检查相同名称的现有表，以确保不会出现名称冲突，并使用 [CREATE TABLE][CREATE TABLE] 语句创建表。然后，此脚本使用 [ALTER TABLE][ALTER TABLE] 语句指定主密钥和表关系。

复制该脚本并将其粘贴到查询窗口中。单击窗口顶部的“运行”以执行该脚本。

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

## <span id="insertData"></span></a>步骤 6：插入数据

打开一个新查询窗口，然后粘贴以下脚本。运行该脚本以插入数据。此脚本使用 [INSERT][INSERT] 语句将值添加到各列。

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
-- Insert data into the Department table.
INSERT INTO dbo.Department (DepartmentID, [Name], Budget, StartDate, Administrator)
VALUES (1, 'Engineering', 350000.00, '2007-09-01', 2);
INSERT INTO dbo.Department (DepartmentID, [Name], Budget, StartDate, Administrator)
VALUES (2, 'English', 120000.00, '2007-09-01', 6);
INSERT INTO dbo.Department (DepartmentID, [Name], Budget, StartDate, Administrator)
VALUES (4, 'Economics', 200000.00, '2007-09-01', 4);
INSERT INTO dbo.Department (DepartmentID, [Name], Budget, StartDate, Administrator)
VALUES (7, 'Mathematics', 250000.00, '2007-09-01', 3);
GO
-- Insert data into the Course table.
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (1050, 'Chemistry', 4, 1);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (1061, 'Physics', 4, 1);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (1045, 'Calculus', 4, 7);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (2030, 'Poetry', 2, 2);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (2021, 'Composition', 3, 2);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (2042, 'Literature', 4, 2);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (4022, 'Microeconomics', 3, 4);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (4041, 'Macroeconomics', 3, 4);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (4061, 'Quantitative', 2, 4);
INSERT INTO dbo.Course (CourseID, Title, Credits, DepartmentID)
VALUES (3141, 'Trigonometry', 4, 7);
GO
-- Insert data into the OnlineCourse table.
INSERT INTO dbo.OnlineCourse (CourseID, URL)
VALUES (2030, 'http://www.fineartschool.net/Poetry');
INSERT INTO dbo.OnlineCourse (CourseID, URL)
VALUES (2021, 'http://www.fineartschool.net/Composition');
INSERT INTO dbo.OnlineCourse (CourseID, URL)
VALUES (4041, 'http://www.fineartschool.net/Macroeconomics');
INSERT INTO dbo.OnlineCourse (CourseID, URL)
VALUES (3141, 'http://www.fineartschool.net/Trigonometry');
--Insert data into OnsiteCourse table.
INSERT INTO dbo.OnsiteCourse (CourseID, Location, Days, [Time])
VALUES (1050, '123 Smith', 'MTWH', '11:30');
INSERT INTO dbo.OnsiteCourse (CourseID, Location, Days, [Time])
VALUES (1061, '234 Smith', 'TWHF', '13:15');
INSERT INTO dbo.OnsiteCourse (CourseID, Location, Days, [Time])
VALUES (1045, '121 Smith','MWHF', '15:30');
INSERT INTO dbo.OnsiteCourse (CourseID, Location, Days, [Time])
VALUES (4061, '22 Williams', 'TH', '11:15');
INSERT INTO dbo.OnsiteCourse (CourseID, Location, Days, [Time])
VALUES (2042, '225 Adams', 'MTWH', '11:00');
INSERT INTO dbo.OnsiteCourse (CourseID, Location, Days, [Time])
VALUES (4022, '23 Williams', 'MWF', '9:00');
-- Insert data into the CourseInstructor table.
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (1050, 1);
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (1061, 31);
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (1045, 5);
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (2030, 4);
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (2021, 27);
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (2042, 25);
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (4022, 18);
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (4041, 32);
INSERT INTO dbo.CourseInstructor(CourseID, PersonID)
VALUES (4061, 34);
GO
--Insert data into the OfficeAssignment table.
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (1, '17 Smith');
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (4, '29 Adams');
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (5, '37 Williams');
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (18, '143 Smith');
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (25, '57 Adams');
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (27, '271 Williams');
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (31, '131 Smith');
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (32, '203 Williams');
INSERT INTO dbo.OfficeAssignment(InstructorID, Location)
VALUES (34, '213 Smith');
-- Insert data into the StudentGrade table.
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2021, 2, 4);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2030, 2, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2021, 3, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2030, 3, 4);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2021, 6, 2.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2042, 6, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2021, 7, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2042, 7, 4);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2021, 8, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (2042, 8, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4041, 9, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4041, 10, null);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4041, 11, 2.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4041, 12, null);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4061, 12, null);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4022, 14, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4022, 13, 4);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4061, 13, 4);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4041, 14, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4022, 15, 2.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4022, 16, 2);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4022, 17, null);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4022, 19, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4061, 20, 4);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4061, 21, 2);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4022, 22, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4041, 22, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4061, 22, 2.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (4022, 23, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1045, 23, 1.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1061, 24, 4);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1061, 25, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1050, 26, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1061, 26, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1061, 27, 3);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1045, 28, 2.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1050, 28, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1061, 29, 4);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1050, 30, 3.5);
INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
VALUES (1061, 30, 4);
GO
</pre></div>

## <span id="QueryDBSysData"></span></a>步骤 7：在 SQL Database 管理门户中查询示例和系统数据

若要检查您的工作，请运行可返回您刚输入的数据的查询。您也可以运行内置存储过程和数据管理视图，这些视图可提供有关 SQL Database 服务器上运行的数据库的信息。

#### <span id="QueryDB"></span></a>查询示例数据

在新查询窗口中，复制并运行以下 Transact-SQL 脚本以检索您刚添加的一些数据。

<div style="width:auto; height:auto; overflow:auto"><pre>
SELECT
Course.Title as &quot;Course Title&quot;
,Department.Name as &quot;Department&quot;
,Person.LastName as &quot;Instructor&quot;
,OnsiteCourse.Location as &quot;Location&quot;
,OnsiteCourse.Days as &quot;Days&quot;
,OnsiteCourse.Time as &quot;Time&quot;
FROM
Course
INNER JOIN Department
ON Course.DepartmentID = Department.DepartmentID
INNER JOIN CourseInstructor
ON Course.CourseID = CourseInstructor.CourseID
INNER JOIN Person
ON CourseInstructor.PersonID = Person.PersonID
INNER JOIN OnsiteCourse
ON OnsiteCourse.CourseID = CourseInstructor.CourseID;
</pre></div>

您应当会看到如下图所示的结果集。

![图像 13][图像 13]

#### <span id="QuerySys"></span></a>查询系统数据

您也可以使用系统视图和内置存储过程来从服务器获取信息。在本教程中，您将只尝试几个命令。

运行以下命令可确定服务器中的可用数据库。

    SELECT * FROM sys.databases  

运行此命令可返回当前连接到服务器的用户列表。

    SELECT user_name(),suser_sname()

运行此存储过程以返回 **School** 数据库中所有对象的列表。

    EXEC SP_help

不要关闭与 **School** 数据库的门户连接。因为几分钟后您需要再次使用该连接。

## <span id="DBLogin"></span></a>步骤 8：创建数据库登录名并分配权限

在 SQL Database 中，您可以使用 Transact-SQL 创建登录名和授予权限。在本课程中，您将使用 Transact-SQL 执行以下三项操作：创建 SQL Server 身份验证登录名、创建数据库用户并通过角色成员身份授予权限。

SQL Server 身份验证登录名用于建立服务器连接。访问 SQL Database 服务器上数据库的所有用户均要通过提供 SQL Server 身份验证登录名和密码来建立连接。

若要创建登录名，您必须先连接到 **master** 数据库。

#### <span id="CreateLogin"></span></a>创建 SQL Server 身份验证登录名

1.  在[管理门户][管理门户]中，选择“SQL Database”、单击“服务器”、选择该服务器，然后单击白色箭头以打开
    服务器页。

    ![图像 5][图像 5]

2.  在“快速启动”页上，单击“管理服务器”以打开一个指向 SQL Database 管理门户的新连接。

3.  输入管理员名称和密码。这是您在创建服务器时所指定的管理员登录名。

    ![图像 20][图像 11]

4.  将在新浏览器窗口中打开 SQL Database 管理门户。单击顶部的“选择数据库”，然后单击 **master**。

    ![图像 14][图像 14]

5.  如果您在该页面上看到类似如下错误，请忽略它。单击“新建查询”以打开一个允许您对 **master** 数据库执行 Transact-SQL 命令的查询窗口。

    ![图像 15][图像 15]

6.  将以下命令复制并粘贴到该查询窗口中。

        CREATE LOGIN SQLDBLogin WITH password='Password1';

7.  运行该命令以创建一个名为“SQLDBLogin”的新 SQL Server 登录名。

#### <span id="CreateDBuser"></span></a>创建数据库用户并分配权限

在您创建 SQL Server 身份验证登录名后，下一步是分配与该登录名关联的数据库和权限级别。通过在每个数据库上创建“数据库用户”可执行此操作。

1.  返回到可连接 **School** 数据库的 SQL Database 管理门户页。如果您关闭了浏览器窗口，请使用上一课程“使用 Transact-SQL 脚本添加数据和架构”中的步骤启动一个到 **School** 数据库的新连接。

    在 SQL Database 管理门户页上，**School** 数据库名称将显示在左上角。

    ![图像 12][图像 12]

2.  单击“新建查询”以打开一个新查询窗口并复制以下语句。

        CREATE USER SQLDBUser FROM LOGIN SQLDBLogin;

3.  运行该脚本。此脚本将基于该登录名创建一个新数据库用户。

    接下来，您将使用 db\_datareader 角色分配权限。分配给此角色的数据库用户可读取数据库中所有用户表中的全部数据。

4.  打开一个新查询窗口，然后输入并运行下一个语句。此语句将运行内置存储过程，用于将 db\_datareader 角色分配给您刚创建的新用户。

        EXEC sp_addrolemember 'db_datareader', 'SQLDBUser';

    现在，您拥有了一个对 **School** 数据库具有只读权限的新 SQL Server 身份验证登录名。使用这些步骤，您可以创建其他 SQL Server 身份验证登录名，以允许对您的数据进行不同级别的访问。

## <span id="ClientConnection"></span></a>步骤 9：从其他应用程序进行连接

现在您已创建一个操作数据库，您可以从 Excel 工作簿连接到该数据库。

#### 从 Excel 进行连接

如果您的计算机上安装了 Excel 2010，则可以使用下列步骤连接到示例数据库。

1.  在 Excel 的“数据”选项卡上，单击“来自其他源”，然后单击“来自 SQL Server”。

2.  在数据连接向导中，输入 SQL Database 服务器的完全限定域名，后跟具有数据库访问权限的 SQL Server 身份验证登录名。

可在“快速链接”下的“数据库”页上找到服务器名称。在 Azure 管理门户上，您可以在 SQL Database“服务器”页的“仪表板”上的“管理 URL”中找到该服务器名称。

服务器名称由一串字母和数字组成，后跟“.database.windows.net”。在数据库连接向导中指定此名称。在指定该名称时不要包含 http:// 或 https:// 前缀。

输入 SQL Server 身份验证登录名。出于测试目的，您可以使用在设置服务器时创建的管理员登录名。若要进行常规数据访问，请使用与您刚创建的数据库用户登录名类似的登录名。

    ![Image16](./media/sql-database-get-started/16ExcelConnect_SQLTut.png)

1.  在下一页上，选择 **School** 数据库，然后选择 **Course**。单击“完成”。

    ![图像 17][图像 17]

2.  此时将显示“导入数据”对话框，用于提示您选择导入数据的方法和位置。选择默认选项后，单击“确定”。

    ![图像 19][图像 19]

3.  在工作表中，您应当会看到与下面类似的表。

    ![图像 18][图像 18]

如果只使用 Excel，那么您一次只能导入一个表。较好的方法是使用 PowerPivot for Excel 外接程序，以便以单个数据集的形式导入和使用多个表。PowerPivot 的使用方法不属于本教程的范围，但您可以在此 [Microsoft 网站][Microsoft 网站]上获取更多信息。
<!-- <h2><a id="ConfigureDataSync"></a>Step 10: Configure SQL Data Sync</h2>  <h4>SQL Data Sync</h4>  Now that you've created your SQL Database instances, you can leverage SQL Data Sync to keep your high value data synchronized across multiple locations.  SQL Data Sync is a feature of SQL Database that enables you to synchronize selected data, either on a schedule or on-demand, without writing any code or scripts. SQL Data Sync supports synchronization between SQL Database instances or hybrid topologies that include SQL Databases and instances of SQL Server.  For more information about SQL Data Sync, see [Getting Started with SQL Data Sync](http://go.microsoft.com/fwlink/?LinkId=274959). -->

## <span id="NextSteps"></span></a>后续步骤

</p>
现在，您已了解 SQL Database 和管理门户，接下来，您可以尝试 SQL Server 数据库管理员使用的其他一些工具和方法。

若要主动管理新数据库，请考虑安装和使用 SQL Server Management Studio。Management Studio 是用于管理 SQL Server 数据库（包括在 Azure 上运行的数据库）的主要数据库管理工具。使用 Management Studio，您可以保存查询以供将来使用、添加新表和存储过程，以及磨练您在丰富脚本环境（包括语法检查程序、智能感知和模板）中掌握的 Transact-SQL 技能。若要开始操作，请按照[使用 SQL Server Management Studio 管理 SQL Database][使用 SQL Server Management Studio 管理 SQL Database] 中的说明操作。

流利使用 Transact-SQL 查询和数据定义语言对数据库管理员而言至关重要。如果您是第一次使用 Transact-SQL，请从[教程：编写 Transact-SQL 语句][教程：编写 Transact-SQL 语句]开始以了解一些基本技能。

也可采用其他方法将本地数据库移动到 SQL Database。如果您拥有现有数据库，或者下载了示例数据库进行练习，请尝试以下替代方法：

-   [将数据库迁移到 SQL Database][将数据库迁移到 SQL Database]
-   [在 SQL Database 中复制数据库][在 SQL Database 中复制数据库]

  [步骤 1：创建 Azure 帐户]: #Subscribe
  [步骤 2：连接到 Azure 并创建数据库]: #Connect
  [步骤 3：配置防火墙]: #ConfigFirewall
  [步骤 4：使用 Transact-SQL 脚本添加数据和架构]: #AddData
  [步骤 5：创建架构]: #createschema
  [步骤 6：插入数据]: #insertData
  [步骤 7：在 SQL Database 管理门户中查询示例和系统数据]: #QueryDBSysData
  [步骤 8：创建数据库登录名并分配权限]: #DBLogin
  [步骤 9：从其他应用程序进行连接]: #ClientConnection
  [管理门户]: http://manage.windowsazure.cn
  [图像 1]: ./media/sql-database-get-started/1NavPaneDBSelected_SQLTut.png
  [图像 2]: ./media/sql-database-get-started/2MainPageCustomCreateDB_SQLTut.png
  [图像 3]: ./media/sql-database-get-started/3DatabaseSettings_SQLTut.PNG
  [图像 4]: ./media/sql-database-get-started/4ServerSettings_SQLTut.PNG
  [SQL Database 防火墙]: http://social.technet.microsoft.com/wiki/contents/articles/2677.sql-azure-firewall-zh-cn.aspx
  [图像 5]: ./media/sql-database-get-started/5DBPortalDatabasesServers_SQLTut.PNG
  [图像 6]: ./media/sql-database-get-started/6DBConfigFirewall_SQLTut.png
  [图像 7]: ./media/sql-database-get-started/7DBConfigFirewallSAVE_SQLTut.png
  [MSDN 文章]: http://msdn.microsoft.com/zh-cn/library/azure/ee621790.aspx "MSDN 文章"
  [图像 8]: ./media/sql-database-get-started/8MainPageHome_SQLTut.png
  [图像 9]: ./media/sql-database-get-started/9DBListSchool_SQLTut.png
  [图像 10]: ./media/sql-database-get-started/10DBPortalManageButton_SQLTut.png
  [图像 11]: ./media/sql-database-get-started/11ManageDatabaseLogin_SQLTut.png
  [图像 12]: ./media/sql-database-get-started/12DBPortalNewQuery_SQLTut.PNG
  [CREATE TABLE]: http://msdn.microsoft.com/zh-cn/library/azure/ee336258.aspx
  [ALTER TABLE]: http://msdn.microsoft.com/zh-cn/library/azure/ee336286.aspx
  [INSERT]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee336284.aspx
  [图像 13]: ./media/sql-database-get-started/13DBQueryResults_SQLTut.PNG
  [图像 14]: ./media/sql-database-get-started/14DBPortalConnectMaster_SQLTut.PNG
  [图像 15]: ./media/sql-database-get-started/15DBPortalConnectMasterErr_SQLTut.PNG
  [图像 17]: ./media/sql-database-get-started/17ExcelSelect_SQLTut.png
  [图像 19]: ./media/sql-database-get-started/19ExcelImport_SQLTut.png
  [图像 18]: ./media/sql-database-get-started/18ExcelTable_SQLTut.PNG
  [Microsoft 网站]: http://www.microsoft.com/zh-cn/server-cloud/solutions/business-intelligence/default.aspx#fbid=9ZH5wGSDgf0
  [使用 SQL Server Management Studio 管理 SQL Database]: http://azure.microsoft.com/zh-cn/documentation/articles/sql-database-manage-azure-ssms/
  [教程：编写 Transact-SQL 语句]: http://msdn.microsoft.com/zh-cn/library/ms365303.aspx
  [将数据库迁移到 SQL Database]: http://msdn.microsoft.com/zh-cn/library/azure/ee730904.aspx
  [在 SQL Database 中复制数据库]: http://msdn.microsoft.com/zh-cn/library/azure/ff951624.aspx
