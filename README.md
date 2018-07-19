## **Workshop DevOps - Windows**

### **Setup do ambiente de desenvolvimento**

1.  **Criar usuário IAM com acesso de administrador.**
    Dentro do console AWS, clique em **Services**, em seguida digite no campo de busca de serviços **IAM** e selecione o serviço **IAM**.
	- Na página do seviço IAM, crie um novo grupo clicando **Groups”**, em seguida clique em **Create New Group**.
	- Nomeie o grupo de acordo com sua finalidade, neste caso "Admin”, e prossiga para o próximo passo.
	- Atribua as permissões de administrador selecionando a policy **AdministratorAccess**, e prossiga.
	- Verifique se as configurações estão corretas e clique em “Create Group”
	    
    Agora vamos criar um usuário e adicioná-lo ao grupo Admin.
	- Clique em **Users**, em seguida clique em **Add User**.
	- Nomeie o novo usuário seguindo o padrão "iam-seunome".
	- Selecione o tipo de acesso do usuário, neste caso selecionaremos ambos os tipo acesso programático e ao gerenciamento do console AWS.
		- Adicione uma senha e prossiga.
		- Selecione o grupo **Admin** criado anteriormente.
		- Revise as cofigurações e clique em **Create User**.
		- Ao final da criação do usuário será exibido na tela as informações de credenciais para acesso programático, baixe o arquivo “.csv”.
        
 2. **Criar chave privada para acesso aos hosts.**
	- Dentro do console AWS, clique em **Services**, em seguida digite no campo de busca de serviços “EC2” e selecione o serviço **EC2**.
	- Selecione a região “Ohio”.
	- No painel lateral esquerdo, procure por **NETWORK & SECURITY** e clique em **Key Pairs**, em seguida clique em **Create Key Pair**.
		- Nomeie a Key Pair seguindo o padrão “key-seunome” e clique em **Create**.
		- Baixe o arquivo **key-seunome.pem**.
        
 3. **Criar instancia EC2  para uso como bastion host (windows 2016) e instalação do Visual Studio.**
 
	- Ainda na página do serviço EC2, no painel à esquerda, procure por **INSTANCES** e clique em **Instances**.
	- Clique no botão **Launch Instance**.
		- No **Step 1: Choose your AMI**, selecione a AMI  de nome **Microsoft Windows Server 2016 Base**.
		- No **Step 2: Choose  an instance type**, selecione t2.medium e prossiga.
		- No **Step 3: Configure instance details**, vamos usar as configurações padrão da **VPC default**, portanto, somente verifique se o campo **Auto-assign Public IP** está definido como "use subnet setting(Enable)".
		- Adicione duas tags seguindo o modelo “chave:valor”, onde por exemplo  **chave = Ambiente** e **valor = DevOps**, sendo a primeira **Ambiente:DevOps** e outra tag definida como **Name:bastion-host**.
		- Nas configurações de **security group**, edite o campo **Source** clicando em **Custom** e altere para **My IP**. Desta forma já limitamos muito as possibilidades de ataques permitindo somente conexões originadas de uma única origem conhecida (obs: em ambientes de produção certifique-se de adicionar todos os endereços IP utilizados pela empresa para acesso à internet). Adicione um nome amigável ao security group de forma que possa reutilizá-lo em regras futuras. Por exemplo, “myip-rdp-allow”.
		- Siga completando o processo até o momento de finalização quando é solicidado a key pair de acesso, selecione a chave criada anteriormente, **key-seunome.pem**. Essa chave será usada para criptografar a senha inicial do usuário **Administrator** do Windows.
		- Volte para o dashboard de instâncias clicando no id da instância e acompanhe a finalização do provisionamento.

4. **Acessar o bastion host.**
	- Após a conclusão da inicialização clique no botão “Actions” e selecione “Connect”.
		- Nesta tela usaremos a key pair “key-seunome” que criamos para descriptografar a senha do “Adminitrator”. Clique em “Get Password”, em seguida em ”Browse” e navegue até o caminho onde salvou o arquivo “key-seunome.pem”. Clique em “Decrypt Password” e copie a senha decriptografada.
		- Baixe o arquivo de configuração com os dados da conexão RDP e abra com seu cliente RDP de preferência. Use a senha descriptografada para ter acesso como “Adminstrator”.
        
5. **Baixar e instalar o visual studio 2017.**
**Dica** - Por padrão as configurações de segurança do IE bloqueaiam o download. Para simplificar e agilizar o processo podemos desabilitar as configurações de segurança do IE abrindo o **Server Manager** do Windows, em seguida selecione **Local Server** e procure por **IE Enhanced Security Configuration** que estará definido como "On", altere para “Off” e feche o **Server Manager**.
	- Após conectado via RDP ao bastion host, abra o navegador e baixe o visual studio 2017 Community Edition no link *https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community&rel=15*
	- Instale o Visual Studio seguindo o processo NNF (não precisa selecionar nenhum pacote adicional), feche o Visual Studio após finalizar a instalação.

6. **Baixar e instalar o aws toolkit para vs2017.**
	- Baixe o AWS ToolKit para Visual Studio 2017 no link *https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.AWSToolkitforVisualStudio2017*
	- Clique no instalador e prossiga com a instalação, durante o processo de instalação permita que os os pacotes adicionais  requeridos pelo aws toolkit sejam instalados. Aguarde a conclusão da instalação que pode levar alguns minutos.

7. **Configurando as credenciais de acesso .**
 	- Ao final da instalação, abra novamente o Visual Studio, estaremos na página do **AWS Gettting Started with Visual Studio 2017**.
	- Preencha os campos solicitados em **Credential Setup** com as informações salvas no arquivo ".csv" que baixamos no início do tutorial quando criamos o usuário IAM.
		- Profile Name - Coloque o nome do usuario IAM “iam-seunome” criado anteriormente.
		- Account Number - Para saber o número da conta, no painel de gerenciamento AWS, clique em **My Account** e observe o valor do campo **Account ID**.
		- Salve as configurações.

### Introdução ao CI/CD


8. **Criando um repositório no  AWS CodeCommit com visual studio 2017**

	Neste ponto, o ambiente do bastion host está pronto e estaremos conectados à ele. Primeiramente precisamos criar um repositório no CodeCommit para sincronizar com repositório local.
	- Dentro do Visual Studio, navegue para a janela **Team Explorer**, clique no ícone **Manage Connections** (parece um conector de tomada), novas opções de repositórios surgirão.
		- Clique em **Connect** na sessão do **AWS CodeCommit**.
			
		Como nosso perfil atual do AWS tool kit possui permissão de Administrador por padrão o perfil atual será selecionado, mas caso tivessemos outros perfis configurados, precisaríamos selecionar o perfil desejado neste momento.
		Após conetar-se ao serviço AWS CodeCommit, as opções de criar/clonar repositórios ou sair estarão disponíveis.

		- Clique em **Create** e preencha as informações solicitadas (nome do repositorio = repo-seunome) e prossiga.
		- Neste ponto será solitidado novas credenciais, essas credenciais são específicas para acesso ao serviço AWS CodeCommit.
			- Entre no console AWS e navegue até a página do serviço IAM, selecione **Users**, e clique no usuário **iam-seunome**, na aba **Security Credentials** navegue até a  seção **HTTPS Git credentials for AWS CodeCommit** e clique em **Generate**. Baixe o arquivo ".csv" contendo as credenciais.
			- Retorne para o Visual Studio e adicione as informações de acesso contidas no arquivo ".csv" criadas no passo anterior.

9. **Baixando o programa de exemplo.**
	Neste laboratório utilizaremos dois exemplos de aplicações .NetCore utilizando o SDK da AWS.
	- Dentro da janela **Team Explorer**, nas opções de repositório **Local** do git, clique em **Clone** e adicione a ULR do repositório de exemplos de aplicações .NET da AWS, *https://github.com/awslabs/aws-sdk-net-samples.git*.
		- Após clonar o repositório, abra uma janela do **Windows Explorer** e navegue até a raíz dos repositórios “C:\Users\Administrator\source\repos”.
		- Copie o conteúdo do diretório *"aws-sdk-net-samples/ConsoleSample/AmazonS3Sample/\*"* e cole dentro do diretório *“repo-seunome/\*”.*
		- Volte para o Visual Studio.
		- Novamente na janela **Team Explorer**, na seção **Solutions**, clique duas vezes na aplicação **AmazonS3Sample.sln** e depois navegue para a janela **Solution Explorer**.
		- Configure o perfil do AWS tool kit criado anteriormente (iam-seunome) nas configurações da aplicação.
			- Dentro da janela **Solution Explorer** abra o arquivo **App.config** e altere a seção **\<appSettings\>** conforme exemplo abaixo e salve as alterações
		   	
			``` 
			<appSettings>
			      <add key=“AWSProfileName” value=“iam-seunome”/>
			</appSettings>
			```

		- Customize as variáveis **bucketName** e **keyName** na aplicação.
		Ainda dentro de **Solution Explorer** abra o arquivo **S3Sample.cs** e edite as linhas de código onde as variáveis são definidas conforme exempo abaixo:
			 
			```
			static string bucketName = “bucket-seunome-suaidade”;
			static string keyName = “file-immersionday”;
			```
	
		- Salve as alterações de perte F5 para testarmos o programa localmente. O programa deverá listar os buckets já existentes no AWS S3, criar um novo bucket com o nome especificado, criar um objeto dentro do novo bucket com o nome definido em **keyName**, ler o objeto recém criado, deletar o objeto e listar o bucket recém criado. 
		- Dentro do Visual Studio, navegue até a janela **AWS Explorer** e clique no ícone de **Refresh**, abra a seção do S3 e procure pelo bucket recém criado.

