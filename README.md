## **Workshop DevOps - Windows**

**Setup do ambiente de desenvolvimento**

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
