#���ʹ����� Mac �� Linux �� Azure �����й���

��ָ�Ͻ������ʹ�������� Mac �� Linux �� Azure �����й��ߴ����͹��� Azure �еķ������漰�ķ�������**��װ����**��**���뷢������**��**�����͹��� Azure ��վ**�Լ�**�����͹��� Azure �����**���й������Ĳο��ĵ��������[������ Mac �� Linux �� Azure �����й����ĵ�][reference-docs]��

##Ŀ¼
* [ʲô�������� Mac �� Linux �� Azure �����й���](#Overview)
* [��ΰ�װ���� Mac �� Linux �� Azure �����й���](#Download)
* [��δ��� Azure �ʻ�](#CreateAccount)
* [������غ͵��뷢������](#Account)
* [��δ����͹��� Azure ��վ](#WebSites)
* [��δ����͹��� Azure �����](#VMs)


<h2><a id="Overview"></a>ʲô�������� Mac �� Linux �� Azure �����й���</h2>

������ Mac �� Linux �� Azure �����й�����һ�����ڲ���͹��� Azure ����������й��ߡ�
 
֧�ֵ����������

* ���뷢�����á�
* �����͹��� Azure ��վ��
* �����͹��� Azure �������

�й�֧������������б�������Щ���߰�װ���������д����� `azure -help`����μ�[�ο��ĵ�][reference-docs]��

<h2><a id="Download">��ΰ�װ���� Mac �� Linux �� Azure �����й���</a></h2>

�����б�����йذ�װ�����й��ߵ���Ϣ������ȡ������Ĳ���ϵͳ����

* **Mac**������ [Azure SDK ��װ����][mac-installer]�������ص� .pkg �ļ���������ʾ��ɰ�װ���衣

* **Linux**����װ���°汾�� [Node.js][nodejs-org]�������[ͨ���������������װ Node.js][install-node-linux]����Ȼ�������������

		npm install azure-cli -g

	**ע��**���������Ҫʹ��������Ȩ�޲������д����

		sudo npm install azure-cli -g

* **Windows**�����д˴� [Azure �����й���][windows-installer]�ṩ�� Windows ��װ����.msi �ļ�����


��Ҫ���԰�װ������������ʾ���¼��� `azure`�������װ�ɹ����㽫�ῴ�����п��� `azure` ������б�

<h2><a id="CreateAccount"></a>��δ��� Azure �ʻ�</h2>

��Ҫʹ�������� Mac �� Linux �� Azure �����й��ߣ�����Ҫһ�� Azure �ʻ���

�� Web ������������ [http://www.azure.cn][azuredotcn] ���������Ͻǵ�**�������**��

![Azure ��վ][Azure Web Site]

��ѭ�йش����ʻ���˵����

<h2><a id="Account"></a>������غ͵��뷢������</h2>

��Ҫ��ʼ����������Ҫ�����ز��������ķ������á��⽫������ʹ����Щ�����������͹��� Azure ������Ҫ������ķ������ã���ʹ�� `account download` ���

	azure account download

�⽫��Ĭ�������������ʾ����¼�������Ż�����¼�󣬽���������� `.publishsettings` �ļ������´��ļ��ı���λ�á�

��������ͨ��������������� `.publishsettings` �ļ������� `{path to .publishsettings file}` �滻Ϊ `.publishsettings` �ļ���·����

	azure account import {path to .publishsettings file}

����ʹ�� <code>account clear</code> ������ɾ��ͨ�� <code>import</code> ����洢��������Ϣ��

	azure account clear

��Ҫ�鿴 `account` �����ѡ���б���ʹ�� `-help` ѡ�

	azure account -help

���뷢�����ú�Ϊ��ȫ�����Ӧɾ�� `.publishsettings` �ļ���

> [AZURE.NOTE] ���뷢������ʱ�����ڷ��� Azure ���ĵ�ƾ�ݽ��洢����� `user` �ļ����С���� `user` �ļ����ܲ���ϵͳ���������ǣ��������ȡ������ʩ������� `user` �ļ��н��м��ܡ����ͨ�����·�ʽ��ɴ˲�����
> 
> - �� Windows �ϣ��޸��ļ������Ի�ʹ�� BitLocker��
> - �� Mac �ϣ�Ϊ�ļ������� FileVault��
> - �� Ubuntu �ϣ�ʹ�á�������Ŀ¼�����ܡ����� Linux �ַ��ṩ�˵�Ч���ܡ�

��ʱ����׼���ô����͹��� Azure ��վ�� Azure �������

<h2><a id="WebSites"></a>��δ����͹��� Azure ��վ</h2>

###������վ

��Ҫ���� Azure ��վ�����ȴ���һ����Ϊ `MySite` �Ŀ�Ŀ¼��Ȼ���������Ŀ¼��

Ȼ�������������

	azure site create MySite --git

�����������������´�������վ��Ĭ�� URL��ѡ�� `--git` ������ͨ���ڱ���Ӧ�ó���Ŀ¼����վ���������Ĵ��� git �洢����ʹ�� git ��������վ����ע�⣬��������ļ������� git �洢�⣬�������Ὣ�µ�Զ����ӵ����д洢�⣬��ָ����վ���������ڵĴ洢�⡣

��ע�⣬��ɽ� `azure site create` ������������һѡ��һ��ִ�У�

* `--location [location name]`����ѡ��������ָ��������վ���������ĵ�λ�ã����硰��������������������Դ�ѡ�ϵͳ����ʾ��ѡ��һ��λ�á�
* `--hostname [custom host name]`����ѡ��������ָ����վ���Զ�����������

Ȼ������Խ�������ӵ���վĿ¼��ʹ�ó��� git ����`git add`��`git commit`�����ύ������ݡ�ʹ������ git ����ɽ���վ�������͵� Azure��

	git push azure master

###���ô� GitHub �ķ���

��Ҫ�� GitHub �洢�������������������ڴ���վ��ʱʹ�� `--GitHub` ѡ�

	auzre site create MySite --github --githubusername username --githubpassword password --githubrepository githubuser/reponame

�����ӵ�� GitHub �洢��ı��ؿ�¡��������ӵ�д��� GitHub �洢��ĵ���Զ�����õĴ洢�⣬���������Զ��� GitHub �洢���еĴ��뷢��������վ�㡣�Դˣ��κ����͵� GitHub �洢��ĸ��Ķ����Զ�����������վ�㡣

�����ô� GitHub �ķ���ʱ����ʹ�õ�Ĭ�Ϸ�֧Ϊ master ��֧����Ҫָ��������֧����ӱ��ش洢��ִ���������

	azure site repository <branch name>

###����Ӧ������

Ӧ��������������ʱ�ɹ�Ӧ�ó���ʹ�õļ�ֵ�ԡ������ Azure ��վ��������ʱ��Ӧ������ֵ����д����վ�� Web.config �ļ��ж���ľ�����ͬ�������á����� Node.js �� PHP Ӧ�ó���Ӧ�����ÿ��������������������ʾ����ʾ������ü�ֵ�ԣ�

	azure site config add <key>=<value> 

��Ҫ�鿴���м�ֵ�Ե��б���ʹ���������

	azure site config list 

���ߣ������֪���ü���ϣ���鿴ֵ�����ʹ�ã�

	azure site config get <key> 

��Ҫ�������м���ֵ���������������м���Ȼ��������Ӹü����������Ϊ��

	azure site config clear <key> 

###�г�����ʾվ��

��Ҫ�г������վ����ʹ���������

	azure site list

��Ҫ��ȡ�й�վ�����ϸ��Ϣ����ʹ�� `site show` ��������ʾ����ʾ�� `MySite` ����ϸ��Ϣ��

	azure site show MySite

###ֹͣ����������������վ��

�����ʹ�� `site stop`��`site start` �� `site restart` ����ֹͣ����������������վ�㣺

	azure site stop MySite
	azure site start MySite
	azure site restart MySite

###ɾ��վ��

������ʹ�� `site delete` ����ɾ��վ�㣺

	azure site delete MySite

��ע�⣬���������� `site create` ���ļ���������������һ��������轫վ������ `MySite` ָ��Ϊ���һ��������

��Ҫ�鿴 `site` ����������б���ʹ�� `-help` ѡ�

	azure site -help 

<h2><a id="VMs"></a>��δ����͹��� Azure �����</h2>

�����ṩ�Ļ�ӳ����п��õ������ӳ��.vhd �ļ������� Azure ���������Ҫ�鿴���õ�ӳ����ʹ�� `vm image list` ���

	azure vm image list

���ʹ�� `vm create` ����ӿ���ӳ��֮һ���ú�����������������ʾ����ʾ����δ�ӳ��� (CentOS 6.2) �е�ӳ�񴴽� Linux ���������Ϊ `myVM`����������ĸ��û���������ֱ�Ϊ `myusername` �� `Mypassw0rd`������ע�⣬`--location` ����ָ�������д�����������������ġ�������� `--location` ��������ϵͳ����ʾ��ѡ��һ��λ�á���

	azure vm create myVM OpenLogic__OpenLogic-CentOS-62-20120509-en-us-30GB.vhd myusername --location "West US"

����Կ��ǽ� `--ssh` ��־ (Linux) �� `--rdp` ��־ (Windows) ���ݵ� `vm create` ��֧�����´������������Զ�����ӡ�

������Ը����Զ���ӳ����������������ʹ�� `vm image create` ����� .vhd �ļ�����ӳ��Ȼ��ʹ�� `vm create` ��������������������ʾ����ʾ����δӱ��� .vhd �ļ����� Linux ӳ����Ϊ `myImage`������`--location` ����ָ�������д洢ӳ������ݡ���

	azure vm image create myImage /path/to/myImage.vhd --os linux --location "West US"

����ԴӴ洢�� Azure Blob �洢�е� .vhd ����ӳ�񣬶����ôӱ��� .vhd ����ӳ�����ʹ�� `blob-url` ����������һ�㣺

	azure vm image create myImage --blob-url <url to .vhd in Blob Storage> --os linux

����ӳ������ʹ�� `vm create` ��ӳ��������������������������洴����ӳ��`myImage`�� ������Ϊ `myVM` ���������

	azure vm create myVM myImage myusername --location "China East"

�������������������Ҫ�����ս������������������Զ�̷��ʣ����������������ʾ��ʹ�� `vm create endpoint` ����� `myVM` �ϵ��ⲿ�˿� 22 �ͱ��ض˿� 22��

	azure vm endpoint create myVM 22 22

���ʹ�� `vm show` �����ȡ�й����������ϸ��Ϣ������ IP ��ַ��DNS ���ƺ��ս����Ϣ����

	azure vm show myVM

��Ҫ�رա������������������������ʹ����������֮һ��

	azure vm shutdown myVM
	azure vm start myVM
	azure vm restart myVM

�����Ҫɾ�� VM����ʹ�� `vm delete` ���

	azure vm delete myVM

�й����ڴ����͹��������������������б���ʹ�� `-h` ѡ�

	azure vm -h

<!-- IMAGES -->
[Azure Web Site]: ./media/crossplat-cmd-tools/freetrial.png

<!-- LINKS -->
[nodejs-org]: http://nodejs.org/
[install-node-linux]: https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager
[mac-installer]: http://go.microsoft.com/fwlink/?LinkId=252249
[windows-installer]: http://go.microsoft.com/fwlink/?LinkID=275464
[reference-docs]: http://go.microsoft.com/fwlink/?LinkId=252246
[azuredotcn]: http://www.azure.cn

<!---HONumber=Mooncake_0307_2016-->