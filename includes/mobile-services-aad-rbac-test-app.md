1. 在 Visual Studio 中，运行客户端应用，并尝试使用我们创建的名为 Dave 的用户帐户进行身份验证。 

    ![](./media/mobile-services-aad-rbac-test-app/dave-login.png)

2. Dave 没有销售组的成员身份。因此，基于角色的访问检查将被拒绝访问表操作。关闭客户端应用。

    ![](./media/mobile-services-aad-rbac-test-app/unauthorized.png)

3. 在 Visual Studio 中，再次运行客户端应用。这次，使用我们创建的名为 Bob 的用户帐户进行身份验证。

    ![](./media/mobile-services-aad-rbac-test-app/bob-login.png)

4. Bob 有销售组的成员身份。因此，基于角色的访问检查将允许访问表操作。

    ![](./media/mobile-services-aad-rbac-test-app/success.png)

