# Configuração de implantação automática de código em EC2 com CodeDeploy (AWS Academy)

## Passo 1: Preparar uma instância do EC2 para receber o deploy

### Criar uma instância EC2

1. Acessar a console do serviço EC2
2. Iniciar a criação de VM através do botão **Launch Instance / Executar instânica**;
3. Especificar um nome para a VM e adicionar a **tag** que será usada pelo CodeDeploy para selecionar a VM onde o código será implantado (ex: usar chave _codedeploy_ e valor _producao_)
4. Escolher o sistema operacional Ubuntu Server 22.04 LTS e tipo t2.micro (Free-tier eligible);
5. Baixar chave de acesso (ou usar uma existente se já tiver a chave em sua máquina local);
6. Na seção **Network settings**, configurar um security group para abrir a porta 8080
7. Na seção **Advance configuration**, atribuir para a VM o **IAM role** disponibilizado pelo AWS Academy (com nome LabInstanceProfile);
8. Conectar via SSH na VM e realizar a instação do agente do CodeDeploy:

<pre>
sudo apt-get update
sudo apt-get install ruby-full ruby-webrick wget -y
cd /tmp
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/releases/codedeploy-agent_1.3.2-1902_all.deb
mkdir codedeploy-agent_1.3.2-1902_ubuntu22
dpkg-deb -R codedeploy-agent_1.3.2-1902_all.deb codedeploy-agent_1.3.2-1902_ubuntu22
sed 's/Depends:.*/Depends:ruby3.0/' -i ./codedeploy-agent_1.3.2-1902_ubuntu22/DEBIAN/control
dpkg-deb -b codedeploy-agent_1.3.2-1902_ubuntu22/
sudo dpkg -i codedeploy-agent_1.3.2-1902_ubuntu22.deb
sudo systemctl list-units --type=service | grep codedeploy
</pre>

8. Verificar se o agente está executando:

<pre>
sudo service codedeploy-agent status
</pre>

## Passo 2: Preparar a aplicação

1. Obter o código fonte de [https://github.com/mvneves/codedeploy-example](https://github.com/miguelxvr/aws-deploy-labs)
2. Gerar um arquivo .zip contendo o conteúdo do repositório e fazer upload em um bucket do S3 (Obs: pode usar a opção de **Download ZIP** do GitHub).

## Passo 3: Configurar o serviço CodeDeploy

### Configurar uma nova aplicação no CodeDeploy

1. Acessar o serviço **CodeDeploy**
2. Acessar a opção **Applications / Aplicativos** no menu lateral e clicar em **Create application / Criar aplicativo**
3. Escolher um nome qualquer para a aplicação, escolher a plataforma **EC2/On-premises / EC2/local** e clicar botão **Create application / Criar aplicativo**

### Configurar um grupo de deploy

1. Acessar a aplicação récem criada e clicar em **Create deployment group / Criar grupo de implantação**
2. Atribuir um nome para o grupo (ex: _producao_)
3. Selecionar o service role disponibilizado pelo AWS Educate: **LabRole**
4. Selecionar o tipo do deploy como **In-place / No local**
5. Escolher a configuração do ambiente como **Amazon EC2 Instances / Instâncias do Amazon EC2** e escolher a mesma tag atribuída para a VM (ex: chave _codedeploy_ e valor _producao_)
6. Escolher a opção **nunca** na etapa **Instalar o agente do AWS CodeDeploy** 
7. Desabilitar a opção **Enable load balancing / Habilitar balanceamento de carga**
8. Clicar em **Create deployment group**

## Passo 4: Fazer deploy da aplicação

1. Navegar novamente para a aplicação criada no serviço CodeDeploy
2. Na aba **Deployments / Implantações**, clicar no botão **Create deployment / Criar implantação**
3. Escolher o **Deployment group / Grupo de implantação** criado anteriormente (ex: _producao_)
4. Escolher a opção de obter o código fonte do S3
5. Especificar o caminho do arquivo .zip no S3 (ex: s3://nome-do-bucket/aplicacao.zip)
6. Na seção **Deployment behavior settings / Configurações adicionais do comportamento de implantação**, marcar a opção _Don't fail the deployment... / Não causar a falha da implantação..._ e a _Overwrite the content
 / Substituir o conteúdo_
7. Clicar em **Create deployment / Criar implantação**
8. Acompanhar o progresso da implantação e, quando concluída, verificar se a aplicação subiu com sucesso (Obs: acessar o IP público da VM usando a porta 8080 pelo browser)

## Referências:

* Create a Service Role for CodeDeploy: https://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-service-role.html
* Create an IAM Instance Profile for Your Amazon EC2 Instances (Console): https://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-iam-instance-profile.html
