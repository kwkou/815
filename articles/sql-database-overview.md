# 如何管理 Azure 上的 SQL Database

本指南演示了如何针对 Azure SQL Database 上的逻辑服务器和数据库实例执行管理任务。

## 什么是 SQL Database？

SQL Database 在 Azure 上提供关系数据库管理服务并且基于 SQL Server 技术。通过使用 SQL Database，您可以轻松地设置和部署数据库实例，并且利用能够为企业级可用性、可缩放性和安全性提供内置数据保护和自愈优势的分布式数据中心。

## 目录

-   [登录 Azure][登录 Azure]
-   [配置 SQL Database][配置 SQL Database]
-   [使用 Management Studio 进行连接][使用 Management Studio 进行连接]
-   [将数据库部署到 Azure][将数据库部署到 Azure]
-   [添加登录名和用户][添加登录名和用户]
-   [缩放 SQL Database 解决方案][缩放 SQL Database 解决方案]
-   [监视逻辑服务器和数据库实例][监视逻辑服务器和数据库实例]
-   [后续步骤][后续步骤]

## <span id="PreReq1"></span></a>登录 Azure

SQL Database 在 Azure 上提供关系数据存储、访问和管理服务。若要使用它，您将需要一个 Azure 订阅。

1.  打开 Web 浏览器并浏览到 <http://www.windowsazure.cn>。若要开始使用免费帐户，请单击右上角的“免费试用”并执行相应步骤。

2.  现已创建您的帐户。一切准备就绪，即可开始。

## <span id="PreReq2"></span></a>创建和配置 SQL Database

接下来，您将逐步了解逻辑服务器的创建和配置过程。在新的 Azure（预览版）管理门户中，您可以先使用已修订的工作流创建数据库，然后再创建服务器。

在本指南中，您将首先创建服务器。如果您有要上载的现成 SQL Server 数据库，您可能更喜欢此方法。

### <span id="createsrvr"></span></a>创建逻辑服务器

1.  在 <http://www.windowsazure.cn> 中进行登录，

2.  单击“SQL Database”，然后单击 SQL Database 主页上的“服务器”。

3.  单击页面底部的“添加”。

4.  在“服务器设置”中，输入一个没有空格的单词作为管理员名称。

    SQL Database 使用 SQL 身份验证进行加密连接。将使用您提供的名称创建一个分配给 sysadmin 固定服务器角色的新 SQL Server 身份验证登录名。

    该登录名不能是电子邮件地址、Windows 用户帐户或 Windows Live ID。SQL Database 不支持声明，也不支持 Windows 身份验证。

5.  提供由大小写值以及数字或符号共同组成的 8 个以上字符的强密码。

6.  选择区域。区域将确定服务器的地理位置。区域不能随意切换，因此要选择一个对此服务器有效的区域。选择一个最靠近您的位置。将 Azure 应用程序和数据库放置在同一区域可以降低出口带宽成本以及减少数据延迟情况。

7.  确保“允许服务”选项处于选中状态，以便您能够使用 SQL Database 管理门户、存储服务以及 Azure 上的其他服务连接到此数据库。

8.  完成后，请单击页面底部的复选标记。

请注意，您没有指定服务器名称。SQL Database 会自动生成服务器名称以确保没有重复的 DNS 条目。服务器名称是一个由 10 个字符组成的字母数字字符串。您不能更改 SQL Database 服务器的名称。

在下一步中，您将配置防火墙以便允许您网络上运行的应用程序通过建立连接来访问相关数据。

### <span id="configFWLogical"></span></a>配置逻辑服务器的防火墙

1.  依次单击“SQL Database”、“服务器”以及您刚创建的服务器。

2.  单击**“配置”**。

3.  复制当前客户端 IP 地址。如果您从某网络进行连接，则为您的路由器或代理服务器侦听的 IP 地址。SQL Database 会检测当前连接所使用的 IP 地址，以便您可以创建一个接受来自该设备的连接请求的防火墙规则。

4.  将 IP 地址粘贴到起始和结束地址范围中。日后，如果您遇到指示该范围太窄的连接错误，则可以编辑此规则来扩大范围。

    如果客户端计算机使用动态分配的 IP 地址，您必须指定较宽范围的地址，使其足以包含分配给您网络中计算机的 IP 地址。从较窄的范围开始，然后仅在您需要时对其进行扩展。

5.  为防火墙规则输入名称，例如，您的计算机或公司的名称。

6.  单击复选标记保存该规则。

7.  单击页面底部的“保存”以完成该步骤。如果没有看到“保存”，请刷新浏览器页面。

现在，您已创建逻辑服务器、允许来自您的 IP 地址的入站连接的防火墙规则以及管理员登录名。在下一步中，您将切换到本地计算机以完成剩余配置步骤。

**注意：**您刚才创建的逻辑服务器是临时的，它将动态托管在数据中心的物理服务器上。如果您删除该服务器，应当事先知道这是一个不可恢复的操作。一定要备份您随后上载到该服务器的所有数据库。

## <span id="PreReq3"></span></a>使用 Management Studio 进行连接

Management Studio 是一种管理工具，让您能够管理单个工作空间中的多个 SQL Server 实例和服务器。如果您已经有了本地 SQL Server 实例，则可同时打开与本地实例和 Azure 上的逻辑服务器的连接，以便并行执行任务。

Management Studio 包括在管理门户中当前不可用的功能，例如语法检查程序以及保存脚本和命名查询以供重复使用的功能。SQL Database 只是一个表格格式数据流 (TDS) 终结点。适用于 TDS 的任何工具，包括 Management Studio，都可以有效地执行 SQL Database 操作。您为本地服务器开发的脚本将在 SQL Database 逻辑服务器上运行。

在下一步中，您将使用 Management Studio 连接到 Azure 上的逻辑服务器。执行此步骤要求您安装了 SQL Server Management Studio 2008 R2 或 2012 版本。如果您在下载或连接到 Management Studio 方面需要帮助，请参阅本网站上的[使用 Management Studio 管理 SQL Database][使用 Management Studio 管理 SQL Database]。

在能够连接之前，有时必须创建防火墙例外，允许本地系统的端口 1433 上的出站请求。安全的计算机通常默认不打开端口 1433。

### <span id="configFWOnPremise"></span></a>配置本地服务器的防火墙

1.  在“高级安全 Windows 防火墙”中，创建新的出站规则。

2.  选择“端口”，指定“TCP 1433”，再指定“允许连接”，并确认“公用”配置文件处于选中状态。

3.  提供一个有意义的名称，例如 *WindowsAzureSQLDatabase (tcp-out) port 1433*。

### <span id="logical"></span></a>连接到逻辑服务器

1.  在 Management Studio 的“连接到服务器”中，确认“数据库引擎”处于选中状态，然后按以下格式输入逻辑服务器名称：*servername*.database.chinacloudapi.cn

您还可在管理门户中的服务器仪表板上的“管理 URL”中获取完全限定服务器名称。

1.  在“身份验证”中，选择“SQL Server 身份验证”，然后输入您在配置逻辑服务器时创建的管理员登录名。

2.  单击“选项”。

3.  在“连接到数据库”中，指定“master”。

### <span id="premise"></span></a>连接到本地服务器

1.  在 Management Studio 的“连接到服务器”中，确认“数据库引擎”处于选中状态，然后按以下格式输入本地实例的名称：*servername*\\*instancename*。如果服务器是本地的，而且是默认实例，请输入 *localhost*。

2.  在“身份验证”中，选择“Windows 身份验证”，然后输入作为 sysadmin 角色成员的 Windows 帐户。

## <span id="HowTo1"></span></a>将数据库部署到 Azure

有很多方法可将本地 SQL Server 数据库移动到 Azure。在此任务中，您将使用“将数据库部署到 SQL Database”向导来上载示例数据库。

School 示例数据库方便简单；其所有对象均与 SQL Database 兼容，因此不需要修改或准备要迁移的数据库。作为新的管理员，请在使用您自己的数据库之前先尝试部署简单的数据库，以了解相关步骤。

**注意：**有关如何准备要迁移到 Azure 的本地数据库的详细说明，请查看“SQL Database 迁移指南”。此外，请考虑下载 Azure 培训工具包。它包含一个实验，演示了用于迁移本地数据库的替代方法。

### <span id="CreateDB"></span></a>在本地服务器上创建 school 数据库

可在 [SQL Database 管理入门][SQL Database 管理入门]中找到用于创建此数据库的脚本。在本指南中，您将在 Management Studio 中运行这些脚本以创建本地版本的 school 数据库。

1.  在 Management Studio 中，连接到本地服务器。右键单击“数据库”、单击“新建数据库”，然后输入 *school*。

2.  右键单击 *school*、单击“新建查询”。

3.  根据教程复制并执行 Create Schema 脚本。

<div style="width:auto; height:300px; overflow:auto"><pre>
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

接下来，复制并执行 Insert Data 脚本。

<div style="width:auto; height:300px; overflow:auto"><pre>
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

现在，您拥有了一个可导出到 Azure 的本地数据库。接下来，您将运行一个可创建 .bacpac 文件、将其加载到 Azure 上并将其导入到 SQL Database 中的向导。

### <span id="DeployDB"></span></a>部署到 SQL Database

1.  在 Management Studio 中，连接到包含您要迁移的数据库的本地 SQL Server 实例。

2.  右键单击您刚创建的 school 数据库、指向“任务”，然后单击“将数据库部署到 SQL Database”。

3.  在“部署设置”中，为该数据库输入一个名称，如 *school*。

4.  单击“连接”。

5.  在“服务器名称”中，输入 10 个字符的服务器名称，后跟 .databases.chinacloudapi.net。

6.  在“身份验证”中，选择“SQL Server 身份验证”。

7.  输入您在创建 SQL Database 逻辑服务器时设置的管理员登录名和密码。

8.  单击“选项”。

9.  在“连接属性”的“连接到数据库”中，键入 **master**。

10. 单击“连接”。此步骤将结束连接规范并返回到该向导。

11. 单击“下一步”，然后单击“完成”以运行该向导。

### <span id="VerifyDBMigration"></span></a>验证数据库部署

1.  在 Management Studio 中，连接到逻辑服务器。如果您已打开一个连接，则可以关闭它，然后打开一个新的连接。现有连接只显示建立该连接时所运行的数据库。

有关如何连接到逻辑服务器的说明，请参阅本文档中的[使用 Management Studio 进行连接][使用 Management Studio 进行连接]。

1.  展开 Databases 文件夹。您应当会在列表中看到 school 数据库。

2.  右键单击 school 数据库，然后单击“新建查询”。

3.  执行以下查询以验证数据可供访问。

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

## <span id="HowTo2"></span></a>添加登录名和用户

部署数据库后，您需要配置登录名和分配权限。在下面的步骤中，您将运行两个脚本。

对于第一个脚本，您将连接到 master 数据库并运行创建登录名的脚本。登录名将用于支持读写操作，并且委托操作任务，例如能够在没有“SA”权限的情况下运行系统查询。

您创建的登录名必须是 SQL Server 身份验证登录名。如果您已经有了使用 Windows 用户标识或声明标识的现成脚本，则该脚本将不在 SQL Database 上运行。

第二个脚本用于分配数据库用户权限。对于此脚本，您将连接到已在 Azure 上装载的数据库。

### <span id="CreateLogins"></span></a>创建登录名

1.  在 Management Studio 中，连接到 Azure 上的逻辑服务器，展开 Databases 文件夹，右键单击“master”，然后选择“新建查询”。

2.  使用以下 Transact-SQL 语句来创建登录名。将 password 替换为有效密码。分别选择每个登录名，然后单击“执行”。为剩余的登录名重复以上操作。

<div style="width:auto; height:auto; overflow:auto"><pre>
-- run on master, execute each line separately
-- use this login to manage other logins on this server
CREATE LOGIN sqladmin WITH password='&lt;ProvidePassword&gt;'; 
CREATE USER sqladmin FROM LOGIN sqladmin;
EXEC sp_addrolemember 'loginmanager', 'sqladmin';

-- use this login to create or copy a database
CREATE LOGIN sqlops WITH password='&lt;ProvidePassword&gt;';
CREATE USER sqlops FROM LOGIN sqlops;
EXEC sp_addrolemember 'dbmanager', 'sqlops';
</pre></div>

### <span id="CreateDBUsers"></span></a>创建数据库用户

1.  展开 Databases 文件夹，右键单击“school”，然后选择“新建查询”。

2.  使用以下 Transact-SQL 语句来添加数据库用户。将 password 替换为有效密码。

<div style="width:auto; height:auto; overflow:auto"><pre>
-- run on a regular database, execute each line separately
-- use this login for read operations
CREATE LOGIN sqlreader WITH password='&lt;ProvidePassword&gt;';
CREATE USER sqlreader FROM LOGIN sqlreader;
EXEC sp_addrolemember 'db_datareader', 'sqlreader';

-- use this login for write operations
CREATE LOGIN sqlwriter WITH password='&lt;ProvidePassword&gt;';
CREATE USER sqlwriter FROM LOGIN sqlwriter;
EXEC sp_addrolemember 'db_datawriter', 'sqlwriter';

-- grant DMV permissions on the school database to 'sqlops'
GRANT VIEW DATABASE STATE to 'sqlops';
</pre></div>

### <span id="ViewLogins"></span></a>查看和测试登录名

1.  在新查询窗口中，连接到“master”并执行以下语句：

        SELECT * from sys.sql_logins;

2.  在 Management Studio 中，右键单击“school”数据库并选择“新建查询”。

3.  在“查询”菜单上，指向“连接”，然后单击“更改连接”。

4.  以 *sqlreader* 的身份登录。

5.  复制并尝试运行以下语句。您应该收到一条错误信息，指出对象不存在。

        INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
        VALUES (1061, 30, 9);

6.  打开第二个查询窗口，并将连接上下文更改为 *sqlwriter*。相同的查询现在应该成功运行。

您现在已经创建和测试了几个登录名。有关详细信息，请参阅[在 SQL Database 中管理数据库和登录][在 SQL Database 中管理数据库和登录]和[使用动态管理视图监视 SQL Database][使用动态管理视图监视 SQL Database]。

## <span id="HowTo3"></span></a>监视逻辑服务器和数据库实例

您在本地服务器上通常使用的监视工具和方法（例如审核登录名、运行跟踪和使用性能计数器）并不适用于 SQL Database。在 Azure 上，您可以使用数据管理视图 (DMV) 来监视数据容量、查询问题和当前连接。

有关详细信息，请参阅[使用动态管理视图监视 SQL Database][使用动态管理视图监视 SQL Database]。

## <span id="HowTo4"></span></a>缩放 SQL Database 解决方案

在 Azure 上，数据库可伸缩性是横向扩展的同义词，在横向扩展时，工作负载将跨数据中心内的多个商品服务器重新分配。横向扩展是一种解决数据容量或性能问题的策略。一个具有高速增长趋势的超大型数据库无论是供部分用户还是供大量用户访问，最终都将需要横向扩展策略。

通过联合可在 Azure 上以最佳方式实现横向扩展。SQL Database 联合基于水平分片，其中一个或多个表按行进行拆分，并被划分到多个联合成员中。

联合并不是解决每个可伸缩性问题的唯一方法。有时，可根据数据的特征或应用程序要求采用更简单的方法。下表按复杂性顺序显示了可能的解决方案。

**增加数据库的大小**

数据库按照固定大小创建，该固定大小受每个版本强加的最大大小制约。对于 Web 版，您可以将数据库大小最大增加到 5 GB。对于企业版，最大数据库大小为 150 GB。增加数据容量的最有效方式是更改版本和最大大小：

     ALTER DATABASE school MODIFY (EDITION = 'Business', MAXSIZE=10GB);

**使用多个数据库和分配用户**

在有限的情况下，您可以创建数据库的副本，然后跨每个数据库分配登录名和用户。在选择联合方式之前，这是一个重新分配工作负载的常见方法。此方法不仅适用于短期使用并且随后会合并到您保存到本地的主要数据库中的数据库，还适用于提供只读数据的解决方案。

**使用联合**

使用 SQL Database 中的联合方法可大大提高可伸缩性和性能。数据库中的一个或多个表按行进行拆分，并被划分到多个数据库（联合成员）中。这种水平分区通常称为“分片”。如果您需要实现缩放、提高性能或管理容量，则该方法在此类主要情形中很有用。

企业版支持联合。有关详细信息，请参阅 [SQL Database 中的联合][SQL Database 中的联合]和 [SQL Database 联合教程 - DBA][SQL Database 联合教程 - DBA]。

**考虑其他存储形式。**

请记住，Azure 支持多种数据存储形式，包括表存储和 Blob 存储。如果您不需要相关功能，则表或 Blob 存储可能会更经济。

## <span id="NextSteps"></span></a> 后续步骤

现在，您已了解有关 SQL Database 管理的基础知识，单击下面的链接可了解如何执行更复杂的管理任务。

-   请参阅 MSDN 上的 [SQL Database][SQL Database]
-   请访问 [SQL Database TechNet WIKI][SQL Database TechNet WIKI]

  [登录 Azure]: #PreReq1
  [配置 SQL Database]: #PreReq2
  [使用 Management Studio 进行连接]: #PreReq3
  [将数据库部署到 Azure]: #HowTo1
  [添加登录名和用户]: #HowTo2
  [缩放 SQL Database 解决方案]: #HowTo4
  [监视逻辑服务器和数据库实例]: #HowTo3
  [后续步骤]: #NextSteps
  [使用 Management Studio 管理 SQL Database]: http://azure.microsoft.com/zh-cn/documentation/articles/sql-database-manage-azure-ssms/
  [SQL Database 管理入门]: http://www.windowsazure.cn/zh-cn/manage/services/sql-databases/how-to-manage-a-sqldb/
  [在 SQL Database 中管理数据库和登录]: http://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx
  [使用动态管理视图监视 SQL Database]: http://msdn.microsoft.com/zh-cn/library/azure/ff394114.aspx
  [SQL Database 中的联合]: http://msdn.microsoft.com/zh-cn/library/azure/hh597452.aspx
  [SQL Database 联合教程 - DBA]: http://msdn.microsoft.com/zh-cn/library/azure/hh778416.aspx
  [SQL Database]: http://msdn.microsoft.com/zh-cn/library/azure/gg619386
  [SQL Database TechNet WIKI]: http://social.technet.microsoft.com/wiki/contents/articles/2267.sql-azure-technet-wiki-articles-index-zh-cn.aspx
