<properties linkid="manage-services-add-a-vm-to-a-virtual-network" urlDisplayName="Add a VM to virtual network" pageTitle="将虚拟机添加到虚拟网络 &ndash; Azure" metaKeywords="" description="本教程将教您如何创建存储帐户和添加到 Azure 虚拟网络的虚拟机 (VM)。" metaCanonical="" services="virtual-machines,virtual-network" documentationCenter="" title="将虚拟机添加到虚拟网络" authors="" solutions="" manager="" editor="" />

# 将虚拟机添加到虚拟网络

<!--SOMEWHERE IN THIS TUTORIAL I NEED TO XREF TO THE OTHER VMACHINE TUTORIAL -->

本教程将指导您完成创建 Azure 存储帐户和添加到[虚拟网络][虚拟网络]的虚拟机 (VM) 的步骤。

本教程假定您之前未使用过 Azure。

<div class="dev-callout">

**重要说明**
如果计划创建虚拟机以安装新 Active Directory 林，请按照[在 Azure 中安装新 Active Directory 林][在 Azure 中安装新 Active Directory 林]中的说明进行操作。

</div>

## 目标

在本教程中，您将学习：

-   [如何创建存储帐户][如何创建存储帐户]

-   [如何创建虚拟机并将其部署到虚拟网络][如何创建虚拟机并将其部署到虚拟网络]

## 先决条件

-   完成以下教程之一：

    -   [在 Azure 中创建虚拟网络][在 Azure 中创建虚拟网络]

        - 或 -

    -   [创建虚拟网络进行跨界连接][创建虚拟网络进行跨界连接]

-   至少具有一个有效的活动订阅的 Windows Live 帐户。

-   [在 Azure 中创建虚拟网络][在 Azure 中创建虚拟网络]或[创建虚拟网络进行跨界连接][创建虚拟网络进行跨界连接]中的以下对象的名称：

    -   为虚拟网络分配的地缘组。

    -   虚拟网络的名称。

    -   子网的名称。

## <a name="CreateStorageAcct">创建存储帐户</a>

1.  在屏幕左下角的 [Azure 管理门户][Azure 管理门户]中创建虚拟网络后，单击“新建”。

    ![NewStorAcct][NewStorAcct]

2.  在导航窗格中，单击“数据服务”、“存储”和“快速创建”。

    ![QuickCreate][QuickCreate]

3.  输入以下信息，然后单击屏幕右下角的复选标记。

-   **URL：**键入 *yourstorage*。

-   **区域/地缘组：**从下拉列表中，选择“YourAffinityGroup”。

-   **启用地域复制：**选中此框。

    ![CreateNewAcct][CreateNewAcct]

1.  在“存储”页上，当过程完成时，“状态”列将显示“联机”。

    ![NewStorAcctCreated][NewStorAcctCreated]

## <a name="CreateVM">创建虚拟机并将其部署到虚拟网络</a>

**创建虚拟机并将其部署到虚拟网络：**

1.  创建存储帐户后，在屏幕左下角单击“新建”。

    ![NewVM][NewVM]

2.  在导航窗格中，依次单击“计算”、“虚拟机”和“从库中”。

    ![FromGallery][FromGallery]

3.  在“VM OS 选择”屏幕上，选择“2012 年 10 月版 Windows Server 2008 R2 SP1”（或可用的最新版），然后单击“下一步”箭头。

    ![VMOS][VMOS]

4.  在“虚拟机配置”屏幕上，输入以下信息，然后单击“下一步”箭头。
    <!-- SHOULD WE TELL USERS TO WRITE DOWN USER NAME AND PASS?? -->

    **提示：**写下用户名和密码，因为它们是您用来登录到新虚拟机的凭据。

-   **虚拟机名称：**键入 *YourVMachine*。

-   **新用户名：**只读。

-   **新密码：**输入强密码。

-   **确认密码：**重新输入密码。

-   **大小：**选择“小”。

    ![VMConfig][VMConfig]

1.  在“虚拟机模式”屏幕上，输入以下信息，然后单击“下一步”箭头。

-   **独立虚拟机：**将此选项保持选定状态。

-   **DNS 名称：**键入 *yourcloudapplication*。

-   **存储帐户：**选择“yourstorage”。

-   **区域/地缘组/虚拟网络：**从下拉列表中，选择“YourVirtualNetwork”。

    ![VMMode][VMMode]

1.  在“虚拟机选项”屏幕上，输入以下信息，然后单击复选标记按钮。此时将创建您的虚拟机。创建完新的虚拟机需要最多 10 分钟。
    <!-- CONFIRM HOW LONG IT CAN TAKE ON AVG FOR VMACHINE TO BE CREATED -->

-   **可用性集：**选择“无”。

-   **虚拟网络子网：**选择“FrontEndSubnet”。

    <div class="dev-callout">

    **说明**
    您应选择至少一个子网且不应选择网关子网。

    </div>

    ![VMOptions][VMOptions]

1.  创建虚拟机后，虚拟机屏幕上的“状态”将为“正在运行”。

    ![VMInstances][VMInstances]

2.  在导航窗格中，单击“所有项目”。您创建的所有对象将与其当前状态一起显示。

    ![AllTab][AllTab]

## 后续步骤

若要在刚创建的虚拟机上为本地 Active Directory 域额外安装一台域控制器，请参阅[在 Azure 虚拟网络中安装副本 Active Directory 域控制器][在 Azure 虚拟网络中安装副本 Active Directory 域控制器]。

## 另请参阅

-   [Azure 虚拟网络][虚拟网络]

-   [使用网络配置文件配置虚拟网络（可能为英文页面）][使用网络配置文件配置虚拟网络（可能为英文页面）]

<!-- LINKS -->

  [虚拟网络]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156007.aspx
  [在 Azure 中安装新 Active Directory 林]: ../active-directory-forest/
  [如何创建存储帐户]: #CreateStorageAcct
  [如何创建虚拟机并将其部署到虚拟网络]: #CreateVM
  [在 Azure 中创建虚拟网络]: /zh-cn/manage/services/networking/create-a-virtual-network/
  [创建虚拟网络进行跨界连接]: /zh-cn/manage/services/networking/cross-premises-connectivity/
  [Azure 管理门户]: http://manage.windowsazure.com/
  [NewStorAcct]: ./media/virtual-networks-add-virtual-machine/VNTut3_01_NewStorageAccount.png
  [QuickCreate]: ./media/virtual-networks-add-virtual-machine/VNTut3_02_StorageAcct_QuickCreate.png
  [CreateNewAcct]: ./media/virtual-networks-add-virtual-machine/VNTut3_03_CreateNewStorageAccount.png
  [NewStorAcctCreated]: ./media/virtual-networks-add-virtual-machine/VNTut3_04_NewStorageAcctCreated.png
  [NewVM]: ./media/virtual-networks-add-virtual-machine/VNTut3_05_NewVM.png
  [FromGallery]: ./media/virtual-networks-add-virtual-machine/VNTut3_06_VM_FromGallery.png
  [VMOS]: ./media/virtual-networks-add-virtual-machine/VNTut3_07_VMOSSelect_Win2008R2.png
  [VMConfig]: ./media/virtual-networks-add-virtual-machine/VNTut3_08_VMConfig.png
  [VMMode]: ./media/virtual-networks-add-virtual-machine/VNTut3_09_VMMode.png
  [VMOptions]: ./media/virtual-networks-add-virtual-machine/VNTut3_10_VMOptions.png
  [VMInstances]: ./media/virtual-networks-add-virtual-machine/VNTut3_11_VMInstances.png
  [AllTab]: ./media/virtual-networks-add-virtual-machine/VNTut3_12_AllTab.png
  [在 Azure 虚拟网络中安装副本 Active Directory 域控制器]: /zh-cn/manage/services/networking/replica-domain-controller/
  [使用网络配置文件配置虚拟网络（可能为英文页面）]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156097.aspx
