>[AZURE.NOTE] 想要在 Visual Studio 连接到 Azure 中国，可以按照 [Azure 应用程序开发说明 - 设置开发计算机](/documentation/articles/developerdifferences/#confdevcomp)描述的那样修改注册表来改变登录环境。
><br/>如果使用的是 Visual Studio 2015 update 2 或以上版本，也可以通过勾选“Enable isolated Azure Active Directory configuration”来添加 Azure 中国的登录环境。
><br/>![enable-isolated-azure-active-directory-configuration](./media/azure-visual-studio-login-guide/enable-isolated-azure-active-directory-configuration.jpg)
><br/>另外，在 Azure 中国，Visual Studio 的 Microsoft Azure Tools 支持的最好的是 2.9.*。其他版本都或多或少有问题，而最新的 3.0，由于 API 版本的问题，很多功能不能使用。