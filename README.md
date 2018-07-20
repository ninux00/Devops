# **Workshop DevOps - Windows**

## **Setup do ambiente de desenvolvimento**

1. ### **Criar usuário IAM com acesso de administrador.**
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
        
2. ### **Criar chave privada para acesso aos hosts.**
	- Dentro do console AWS, clique em **Services**, em seguida digite no campo de busca de serviços “EC2” e selecione o serviço **EC2**.
	- Selecione a região “Ohio”.
	- No painel lateral esquerdo, procure por **NETWORK & SECURITY** e clique em **Key Pairs**, em seguida clique em **Create Key Pair**.
		- Nomeie a Key Pair seguindo o padrão “key-seunome” e clique em **Create**.
		- Baixe o arquivo **key-seunome.pem**.
        
3. ### **Criar instancia EC2  para uso como bastion host (windows 2016) e instalação do Visual Studio.**
 
	- Ainda na página do serviço EC2, no painel à esquerda, procure por **INSTANCES** e clique em **Instances**.
	- Clique no botão **Launch Instance**.
		- No **Step 1: Choose your AMI**, selecione a AMI  de nome **Microsoft Windows Server 2016 Base**.
		- No **Step 2: Choose  an instance type**, selecione t2.medium e prossiga.
		- No **Step 3: Configure instance details**, vamos usar as configurações padrão da **VPC default**, portanto, somente verifique se o campo **Auto-assign Public IP** está definido como "use subnet setting(Enable)".
		- Adicione duas tags seguindo o modelo “chave:valor”, onde por exemplo  **chave = Ambiente** e **valor = DevOps**, sendo a primeira **Ambiente:DevOps** e outra tag definida como **Name:bastion-host**.
		- Nas configurações de **security group**, edite o campo **Source** clicando em **Custom** e altere para **My IP**. Desta forma já limitamos muito as possibilidades de ataques permitindo somente conexões originadas de uma única origem conhecida (obs: em ambientes de produção certifique-se de adicionar todos os endereços IP utilizados pela empresa para acesso à internet). Adicione um nome amigável ao security group de forma que possa reutilizá-lo em regras futuras. Por exemplo, “myip-rdp-allow”.
		- Siga completando o processo até o momento de finalização quando é solicidado a key pair de acesso, selecione a chave criada anteriormente, **key-seunome.pem**. Essa chave será usada para criptografar a senha inicial do usuário **Administrator** do Windows.
		- Volte para o dashboard de instâncias clicando no id da instância e acompanhe a finalização do provisionamento.

4. ### **Acessar o bastion host.**
	- Após a conclusão da inicialização clique no botão “Actions” e selecione “Connect”.
		- Nesta tela usaremos a key pair “key-seunome” que criamos para descriptografar a senha do “Adminitrator”. Clique em “Get Password”, em seguida em ”Browse” e navegue até o caminho onde salvou o arquivo “key-seunome.pem”. Clique em “Decrypt Password” e copie a senha decriptografada.
		- Baixe o arquivo de configuração com os dados da conexão RDP e abra com seu cliente RDP de preferência. Use a senha descriptografada para ter acesso como “Adminstrator”.
        
5. ### **Baixar e instalar o visual studio 2017.**

	**Dica** - Por padrão as configurações de segurança do IE bloqueaiam o download. Para simplificar e agilizar o processo podemos desabilitar as configurações de segurança do IE abrindo o **Server Manager** do Windows, em seguida selecione **Local Server** e procure por **IE Enhanced Security Configuration** que estará definido como "On", altere para “Off” e feche o **Server Manager**.
	- Após conectado via RDP ao bastion host, abra o navegador e baixe o visual studio 2017 Community Edition no link *https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community&rel=15*
	- Instale o Visual Studio seguindo o processo NNF (não precisa selecionar nenhum pacote adicional), feche o Visual Studio após finalizar a instalação.

6. ### **Baixar e instalar o aws toolkit para vs2017.**
	- Baixe o AWS ToolKit para Visual Studio 2017 no link *https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.AWSToolkitforVisualStudio2017*
	- Clique no instalador e prossiga com a instalação, durante o processo de instalação permita que os os pacotes adicionais  requeridos pelo aws toolkit sejam instalados. Aguarde a conclusão da instalação que pode levar alguns minutos.

7. ### **Configurando as credenciais de acesso .**
 	- Ao final da instalação, abra novamente o Visual Studio, estaremos na página do **AWS Gettting Started with Visual Studio 2017**.
	- Preencha os campos solicitados em **Credential Setup** com as informações salvas no arquivo ".csv" que baixamos no início do tutorial quando criamos o usuário IAM.
		- Profile Name - Coloque o nome do usuario IAM “iam-seunome” criado anteriormente.
		- Account Number - Para saber o número da conta, no painel de gerenciamento AWS, clique em **My Account** e observe o valor do campo **Account ID**.
		- Salve as configurações.

## Introdução ao Continuous Integration

8. ### **Criando um repositório no  AWS CodeCommit com visual studio 2017**

	Neste ponto, o ambiente do bastion host está pronto e estaremos conectados à ele. Primeiramente precisamos criar um repositório no CodeCommit para sincronizar com repositório local.
	- Dentro do Visual Studio, navegue para a janela **Team Explorer**, clique no ícone **Manage Connections** (parece um conector de tomada), novas opções de repositórios surgirão.
		- Clique em **Connect** na sessão do **AWS CodeCommit**.
			
		Como nosso perfil atual do AWS tool kit possui permissão de Administrador por padrão o perfil atual será selecionado, mas caso tivessemos outros perfis configurados, precisaríamos selecionar o perfil desejado neste momento.
		Após conetar-se ao serviço AWS CodeCommit, as opções de criar/clonar repositórios ou sair estarão disponíveis.

		- Clique em **Create** e preencha as informações solicitadas (nome do repositorio = repo-seunome) e prossiga.
		- Neste ponto será solitidado novas credenciais, essas credenciais são específicas para acesso ao serviço AWS CodeCommit.
			- Entre no console AWS e navegue até a página do serviço IAM, selecione **Users**, e clique no usuário **iam-seunome**, na aba **Security Credentials** navegue até a  seção **HTTPS Git credentials for AWS CodeCommit** e clique em **Generate**. Baixe o arquivo ".csv" contendo as credenciais.
			- Retorne para o Visual Studio e adicione as informações de acesso contidas no arquivo ".csv" criadas no passo anterior.

9. ### **Baixando o programa de exemplo.**
	
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



10. ### **Sincronizando repositório local com repositório AWS CodeCommit**

	- Dentro da janela **Team Explorer** clique no ícone **Manage Connections** (parece uma tomada), navegue até a seção do **AWS CodeCommit** e selecione o repositório previamente criado **repo-seunome**.
		- Clique em **Changes** adicione algum comentário e clique em **Commit All**. Desta forma fazemos o commit localmente de todo o conteúdo que copiamos  do diretório “AmazonS3Sample”.
		- Clique no ícone **Home** (parece uma casa :P) e em seguida em **Sync**. Nesta tela é possivel ver os commits pendentes em ambas as direções.
		- Na seção **Outgoing Commits** selecione o commit que acabamos de fazer localmente e em seguida clique em **Push** para enviar o código para a nuvem.
		- No console AWS, navegue até o serviço **CodeCommit**.
		- Clique no nome do seu repositório para explorar as opções.

11. ### **Criando um branch, comparando branches e submetendo um pull request**

	Volte para o Visual Studio.
	- Dentro da janela **Team Explorer**, clique no ícone **Home** e em seguida em **Branches** e então em **New branch**.
	- Nomeie o novo branch, por exemplo “master.v1".
	- Escolha o branch de origem, neste caso “master” e clique em create.

	Perceba que no canto inferior direito da janela do Visual Studio aparece o nome do repositório atual e o branch que estamos trabalhando. Neste local é possivel gerenciar os repositórios e branches atuais entre outras funções.
	
	- Defina o novo branch como o ambiente de trabalho atual.
	- Na janela **Team Explorer**, na seção **Solutions** abra a solução **AmazonS3SAmple.sln**.
	- Na janela **Solutions Explorer**, edite o arquivo **S3sample.cs** e altere o valor da variável **keyName** para “immersionday-file.v1” e salve com **ctrl+s**.
	- Clique com o botão direito no arquivo **S3Sample.cs** e em seguida clique em **Commit**.
		- A janela do **Team explorer** será selecionada já na tela do commit que estamos submetendo, adicione um comentário e clique em **Commit All**.
		- Clique em **Sync** no pop-up e então na tela seguinte clique em **Push** na seção **Outgoing Commits** para enviar as alterações para nuvem.
	- Dentro do console AWS, navegue até o serviço **AWS CodeCommit**, selecione seu repositório e clique em **Branches**. Aqui é possível definir qual é o branch **default** do repositório.
		- Na linha do nome do novo branch **master.v1**, clique em **Compare**.
		- Na tela de comparação de branches, o branch **master** e o branch **master.v1** serão comparados e poderemos visualizar as alterações entre os branches.
			- Clique em **Split** para comparar as alterações em uma nova visualização.
			- Adicione um comentário sobre as diferenças e salve.
	- Clique em **Create Pull Request**.
		- Na tela de requisição, caso o código de origem e destino possam ser mesclados (o CodeCommit verifica se não há conflitos entre os códigos) poderemos submeter a requisição.
		- Adicione um título e uma descrição para a requisição  e clique em **Create**.

12. ### **Desafio**
	- Crie um novo repositório local e clone este repositório *https://github.com/filomenka/fnemec-aws-lab-29-30-05*
	- Baixe e instale o SDK do .NetCore 2.0  conforme link *https://www.microsoft.com/net/learn/get-started/windows*
	- Teste a aplicação localmente com o VS2017.
	- Crie um novo repositório no AWS CodeCommit com nome "repo-immersionday” e envie o código recém clonado para a núvem.

13. ### **Publicando a aplicação no AWS Elastic Beanstalk através do VS2017**
	- Dentro do Visual Studio 2017, na janela **Solutions Explorer**, clique com o botão direto sobre a solução **AspNetCoreWebApplication** que clonamos no passo anterior e em seguida clique em **Publish to AWS Elastic Beanstalk**.
        - Na tela de publicação, defina o perfil de conta AWS **iam-seunome** e a região desejada, no caso **Ohio**.
		- Selecione a opção **Create a new application environment** e prossiga.
		- Na tela **Application Environment**, defina:
			- Um nome para a nova aplicação, por exemplo **AspNetCoreWebApplication**.
			- Selecione o environment, por exemplo **AspNetCoreWebApplication-dev**.
			- Crie um nome para o endpoint do aplicação no Elastic Beanstalk, verifique a disponibilidade e uma vez que o nome esteja disponível prossiga.
		- Na tela **Amazon EC2 Launch Configuration**:
		- No campo **Container type** selecione a imagem **64bit Windows Server 2016 v1.2.0 running IIS 10.0**.
		- No campo **Instance type** selecione **t2.large**.
		- No campo **Key pair** selecione a chave criada inicialmente neste tutorial **key-seunome** e prossiga.
	- Na tela **Permissions** selecione as opções de role padrão para que o Elastic Beanstalk tenha as permissões necessárias para fazer o deploy da aplicação e prossiga.
	- Na tela **Application Options**, mantenha as configurações padrão e prossiga.
	- Na próxima tela revise as configurações e clique em **Deploy**.
	- Aguarde a conclusão do provisionamento do ambiente e acesse a URL da aplicação fornecida pelo Elastic Beanstalk para visualizar a aplicação rodando na nuvem.

14. ### **Criar uma AMI customizada e uma nova instância para instalarmos o Jenkins**
	- Acesse o console de gerenciamento AWS e navegue até o serviço **EC2**.
	- Selecione a instância que criamos manualmente no início deste tutorial, clique em **Actions**, em seguida clique em **Instance State** e então em **Stop**.
	- Aguarde o desligamento da instância e novamente, clique em **Actions**, em seguida clique em **Image** e então em **Create image**.
	- Na tela de criação de imagem, adicione um nome (como "w16_vs17_dotnet20") para a nova AMI e clique em **Create Image**.
	- Crie uma nova instância a partir da AMI recém criada, ela estará disponivel na lista de **My AMIs**. Lembre-se de utilizar o mesmo security group e key pair criados anteriomente neste tutorial.
                
15. ### **Instalando o servidor Jenkins na instância EC2**
	- Depois de conectar-se na nova instância (use mesma senha da instância bastion-host no momento da criação da AMI), faça o download do instalador do Jenkins para Windows disponível em *https://jenkins.io/download* - versão utilizada **Jenkins 2.121.1 LTS**.
	- Execute o instalador e instale o jenkins com as configurações padrão, ao final do processo de instalação uma página web será aberta no endereço *http://localhost:8080* (aguarde um momento e atualize a página, o servidor web está inicializando … :P) informando um arquivo contendo a senha do usuário **Administrador** do Jenkins, abra o arquivo, copiei e cole a senha no local indicado na página web e prossiga.
	- Na página seguinte selecione a opção **Select plugins to install** e prossiga.
		- Na página de seleção de plugins, analise as opções e na seção **Build Tools** remova os plugins “Ant” e “Gradle” e prossiga. No final do processo você deverá criar um novo usuário e senha para gestão do Jenkins.

16. ### **Instalando e configurando o AWS CodePipeline Plugin em um novo projeto**
	- Inicie o Jenkins e na página inicial, selecione **Manage Jenkins**. 
	- Na página **Manage Jenkins**, selecione **Manage plug-ins**. 
	- Selecione a guia **Available** e na caixa de pesquisa **Filter**, digite "AWS CodePipeline". Selecione o **AWS CodePipeline Plugin for Jenkins** na lista e então, clique no botão **Download now and install after restart**. 
	- Na página **Install plug-ins/updates**, selecione a check-box **Restart Jenkins when installation is complete and no jobs are running**. Isso reiniciará o Jenkins, uma outra forma de reiniciar o serviço Jenkins é realizar uma chamada Rest no servidor local para *http://localhost:8080/safeRestart*.
	- Após o restart do Jenkins autentique-se novamente. 
	- Na página principal, selecione **Novo item**.
		- Em **Item name**, digite um nome para o projeto Jenkins (por exemplo, “immertionday-project”). Selecione **Free style project** e depois **OK**. 
	- Na página de configuração do projeto:
		- Na seção **General**, selecione a caixa de seleção **Execute concurrent builds if necessary**.
		- Na seção **Source Code Management**, selecione **AWS CodePipeline**.
			- Selecione a região de **Ohio** (US-EAST-2).
			- Deixe as configurações de proxy em branco.
			- Adicione as credenciais (AccessKey SecretAccesKey) do usuário **iam-seunome** contidas no arquivo ".csv" criado anteriormente neste tutorial.
			- Selecione a opção **Clear workspace before copying**.
			- Na seção **CodePipeline action type**:
				- No campo **Category** selecione **Build**. 
				- No campo **Provider** adicione um nome para identificar este servidor Jenkins durante a configuração do AWS CodePipeline, por exemplo **Jenkins-w16**.
				- No campo **Version** deixe o valor padrão **1**.
			- Na seção **Build Triggers**, selecione **Poll SCM**.
				- No campo **Schedule** adicione uma instrução de periodicidade no mesmo formato do cron, neste exemplo queremos a execução da trigger a cada minuto, portanto adicione 5 "*" separados por espaços, conforme abaixo:
				```
				* * * * *
				```
			- Na seção **Build Enviroment**, não adicione nenhuma opção.
			- Na seção **Build**, clique em **Add build step** e selecione **Execute windows batch commands**.
				- No campo **Commands** adicione os comandos abaixo:
				```
				dotnet restore AspNetCoreWebApplication/AspNetCoreWebApplication.csproj 
				dotnet restore AspNetCoreWebApplicationTest/AspNetCoreWebApplicationTest.csproj
				dotnet publish -c release -o ./build_output AspNetCoreWebApplication/AspNetCoreWebApplication.csproj
				dotnet publish -c release -o ./test_output AspNetCoreWebApplicationTest/AspNetCoreWebApplicationTest.csproj
				dotnet vstest AspNetCoreWebApplicationTest/test_output/AspNetCoreWebApplicationTest.dll
				```

			- Na seção **Post-build Actions**, clique no botão **Add post-build actions** e selecione **AWS CodePipeline Publisher**.
			- Nas opções do **AWS CodePipeline Publisher**, clique em **Add** para publicar um diretório específico do seu projeto, no nosso caso, o diretório **build_output** conterá nossa aplicação compilada após a execução dos comandos na seção de **Build**.
				- No campo **Location**, adicione **AspNetCoreWebApplication/build_output**.
				- No campo **Artifact Name**, adicione um nome (por exemplo **MyAppBuilt**) para o pacote que será produzido a partir do conteúdo especificado em **Location**. Este nome de artefato também será utilizado durante a configuração do AWS CodePipeline na fase de integração com Jenkins.
			- Selecione **Salvar** para salvar seu projeto de build. 

## **Introdução ao Continuous Delivery**

17. ### **Criando um Pipeline de desenvolvimento**

	Neste momento já temos todos os componentes para a execução de um processo de integração e entrega contínua, temos nossa origem de código, temos o servidor Jenkins configurado para compilar e testar o código e temos um ambiente provisionado no AWS Elastic Beanstalk para hospedar nossa aplicação. Agora precisamos de um componente para orquestrar todo o fluxo durante o processo, para isso utilizaremos o AWS CodePipeline. Dentro do console de gerenciamento AWS navegue até o serviço **AWS CodePipeline** e clique em **Get started**:
	- Step 1 - Name: Nomeie o pipeline, como por exemplo **pipe-immersionday**.
	- Step 2 - Source: Selecione o local de origem que o AWS CodePipeline utilizará como ponto de partida para processo de integração e entrega contínua. Neste caso, selecione a opção **AWS CodeCommit**, escolha o repositório **repo-immersionday** e o branch **master** e prossiga.
	- Step 3 - Build: em **Build Provider**, selecione **Add Jenkins**.
		- Na seção **Add Jenkins**, em:
			- **Provider Name**, adicione o mesmo nome configurado no Jenkins durante a configuração do plugin AWS CodePipeline, neste caso **Jenkins-w16**.
			- **Server URL**, adicione o endereço DNS público da instância EC2 que você instalou o Jenkins e adicione a porta 8080, algo neste formato *http://ec2-20-190-228-174.us-east-2.compute.amazonaws.com:8080*
			- **Project Name**, adicione o nome do projeto criado no Jenkins anteriormente, neste caso **immertionday-project** e prossiga.
	- Step 4 - Deploy: no campo **Deployment Provider**, selecione **AWS Elastic Beanstalk**.
		- Na seção **AWS Elastic Beanstalk**, secione a aplicação e o ambiente criados anteriormente ao publicar nossa aplicação através do VS2017, neste caso:
			- No campo **Application name**: adicione **AspNetCoreWebApplication**.
			- No campo **Environment name**: adicione **AspNetCoreWebApplication-env** e prossiga.
	- Step 5 -  AWS Service Role: no campo **Role Name** selecione a opção **AWS-CodePipeline-Service** e prossiga.
	- Step 6 - Revise as configurações do pipeline e clique em **Create Pipeline**.
	- Step 7 - Clique em **Release change** na tela do pipeline recém criado.
	- Step 8 - Faça uma alteração na aplicação e acompanhe a execução do pipeline, desde o commit, passando pelo processo de build no Jenkins (aproveite para ver os logs e as informações de pooling de novas tarefas do plugin do codepipeline) até o deploy no EB.
		
18. ### **Criando um projeto com AWS CodeBuild**

	Neste passo iremos configurar o processo de build da aplicação utilizando o serviço **AWS CodeBuild**. Para isso, dentro do painel de gerenciamento da AWS, navegue até o serviço **AWS CodeBuild** e clique em **Get Started** e preencha os campos conforme abaixo:
	- Step 1: Configure Project
		- Seção **Configure your project**
			- Project name: "immersionday-project"
		- Seção **Source: What to build** 
			- Source provider: Selecione **AWS CodeCommit**
			- Repository: Selecione **repo-immersionday**
			- Git clone depth: Selecione **1**
			- Build Badge: **Assinale esta opção** - a caixa de seleção Build Badge serve para se certificar de que o status de compilação do projeto seja visível e incorporável.
 		- Seção **Environment: How to build** 	
			- Environment image: Selecione **Use an image managed by AWS CodeBuild**
			- Operating system: Selecione **Windows Server**
			- Runtime: Selecione **Base**
			- Runtime version: Selecione **aws/codebuild/windows-base:1.0**
			- Build Specification: Selecione **Insert build commands** e abaixo de **Build commands** clique na opção **Switch do editor** e observe as opções de configurações de exemplo do arquivo **buildspec.yml** que serão exibidas, após isso apague todo o conteúdo e adicione a linhas abaixo:
			
			```
			version: 0.2
			phases:
			  pre_build:
			    commands:
			      - dotnet restore AspNetCoreWebApplication/AspNetCoreWebApplication.csproj
			      - dotnet restore AspNetCoreWebApplicationTest/AspNetCoreWebApplicationTest.csproj
			  build:
			    commands:
			      - dotnet publish -c release -o ./build_output AspNetCoreWebApplication/AspNetCoreWebApplication.csproj
			      - dotnet publish -c release -o ./test_output AspNetCoreWebApplicationTest/AspNetCoreWebApplicationTest.csproj
			  post_build:
			    commands:
			      - dotnet vstest AspNetCoreWebApplicationTest/test_output/AspNetCoreWebApplicationTest.dll
			artifacts:
			  files:
			    - '**/*'
			  base-directory: AspNetCoreWebApplication/build_output
			```
			
			
			- Certificate: Selecione **Do not install any certificate**
		- Seção **Artifacts: Where to put the artifacts from this build project**	
			- Type: Selecione **Amazon S3**
			- Name: “artefato.zip”
			- Path: “artefato.zip”
			- Namespace type: Selecione **Build ID**
			- Bucket name: Selecione o mesmo bucket criado pelo codepipeline, algo parecido com **codepipeline-us-east-2-...**
			- Artifacts packaging: Selecione **Zip**
		- Seção **Cache**
			- Type: Selecione **No cache**
		- Seção **Service role**
			- Selecione **Create a service role in your account**
			- Role Name: Se você seguiu os exemplos de nomes sugeridos neste tutorial o valor **codebuild-immersionday-project-service-role** deverá estar preenchido.
			- Clique em **Continue** 
	- Step 2: **Review** 
		- Dê uma olhada nas configurações e clique em **Save and Build**.
		- Na tela de seleção do repositório para:
			- Branch: Selecione **master**, em seguida clique em **Start build** e acompanhe a execução do processo de compilação nos logs.

19. ### **Integrando o AWS CodeBuild em nosso pipeline**

	Para alterarmos o processo de build e ao invés de utilizarmos o Jenkins, usarmos o projeto do CodeBuild recém criado, precisamos editar o pipeline.
	Dentro do console de gerenciamento AWS, navegue até o serviço **AWS Code Pipeline**, selecione o pipeline **pipe-immersionday** e clique no botão **Edit**.
	Dentro da caixa da fase **Build**, no canto superior direito clique no ícone **lápis** para editar esta seção. E em seguida clique no ícone **+** paralelo ao **Jenkins-w16** para adicionar uma nova ação.
	- Na tela **Add Action** preencha conforme abaixo:
		- Action category: **Build**
		- Action Name: **Codebuild**
		- Build Provider: Selecione **AWS CodeBuild**
		- Na seção **Configure your project**, selecione **Select an existing build project**
		- Project name: Selecione o projeto recém criado no AWS CodeBuild **immersionday-project**
		- Na seção **Input artifacts**, para:
			- Input artifacts 1: selecione **MyApp**
			- Output artifact 1: selecione **MyAppBuilt**
		- Clique em **Add Action** para concluir o processo.
	- Por fim ainda na caixa da fase **Build** clique no ícone **X** da Action **Jenkins-w16** para removermos a ação de bulid com Jenkins, deixando somente o CodeBuild.
	- Clique em **Save pipeline changes** e confirme.
	- Clique em **Release Changes** para testar o pipeline e acompanhe a execução. 

20. ### **Desafio**
	- Adicione um arquivo buildspec.yml no diretório raiz da aplicação contendo os comandos para o build da aplicação e edite o projeto no console do AWS CodeBuild para usar o arquivo e não as instruções adicionadas via console de gerenciamento nos passos anteriores.
	- Através do bastion host edite, edite o index faça o commit da aplicação web para que a mensagem exibida na página seja “You just created a ASP.Net Core web application and published it using CodePipeline integrated with CodeBuild and Elastic Beanstalk!”


aws-windows-deployment-manifest.json

```
{
  "manifestVersion": 1,
  "deployments": {

    "aspNetCoreWeb": [
      {
        "name": "app",
        "parameters": {
          "appBundle": ".",
          "iisPath": "/",
          "iisWebSite": "Default Web Site"
        }
      }
    ]
  }
}
```
