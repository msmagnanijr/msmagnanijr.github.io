---
layout: post
title: DevOps - Uma Abordagem Prática - Capítulo II
modified: 2015-10-22
tags: [DevOps, Vagrant, KVM, Ansible, CI, CD, JBossm Jacoco, JUnit]
comments: true
---

##  JBoss Forge

JBoss Forge é um framework para criação de projetos baseado em padrões utilizando linha de comando. O JBoss Forge cria aplicações baseadas em Java EE em poucos instantes. A idéia é realmente facilitar a vida do desenvolvedor. Mais informações podem ser encontradas na página do projeto [JBoss Forge](http://forge.jboss.org).

O JBoss Forge será utilizado para criar uma pequena aplicação baseada em Maven onde serão aplicadas as práticas de DevOps. O JBoss Developer Studio já possui integração nativa com o JBoss Forge, facilitando assim a criação dessa aplicação. Ele é totalmente baseado no Eclipse e pode ser substituído por uma combinação de Eclipse + JBoss Tools.

Você pode baixar o JBoss Developer Studio gratuitamente na [comunidade JBoss](http://www.jboss.org/products/devstudio/download)

O JBoss Forge cria aplicações baseadas em Maven. Ralize a instalação do Maven no  próximo tópico.


###  Maven: Gerenciamento de Dependências


Apesar do Maven ser um framework voltado para área de desenvolvimento, é de grande importância que o time de operações tenha pelo menos um noção de qual a responsabilidade desse software.

o Maven é um  framework que descreve a aplicação e suas dependências (bibliotecas). Além da descrição, é responsável pelo build das aplicações, compilando o código fonte, executando testes unitários, realizando o empacotamento, entre outros. Basicamente, ele funciona como um substituto do framework Ant e será amplamente utilizado pelo Jenkins.
 
A instalação consiste, basicamente do download do ZIP, seguido da descompactação. O instalador pode ser obtido em  [http://maven.apache.org](http://maven.apache.org).

Por fim, os comandos Maven serão sempre executados pelo intermédio de outras ferramentas como por exemplo Eclipse e Jenkins. Porém, caso haja a necessidade da execução manual em linha de comando, pode-se apontar a variável de ambiente PATH para a instalação do Maven.

### Desenvolvendo com JBoss Forge

Utilizando o JBoss Developer Studio, navegue até `Windows → Show View → Forge Console` e inicie o Forge utilizando o botão start. Para criação de um novo projeto chamado `judcon-brasil`. execute o comando:

{% highlight bash %}
project-new --named judcon-brasil
{% endhighlight %}

Ps: JUDCon = JBoss Users and Developers Conference

![](/images/201510-devops-02.png)

Crie uma nova entidade JPA que representará o `Palestrante` do `JUDCon`:

{% highlight bash %}
jpa-new-entity --named Palestrante
{% endhighlight %}

O proximo passo é adicionar informações relacionadas ao `Palestrante` como: Bio, Nome, etc. Execute os comandos abaixo:

{% highlight bash %}
[Speaker.java]$ jpa-new-field --named nome
[Speaker.java]$ jpa-new-field --named sobrenome
[Speaker.java]$ jpa-new-field --named bio --length 2000
[Speaker.java]$ jpa-new-field --named twitter
{% endhighlight %}

Observe que a entidade `Palestrante` possui todas as informações necessárias e está pronta para uso:

{% highlight java %}
@Entity
public class Palestrante implements Serializable {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "id", updatable = false, nullable = false)
  private Long id;
  @Version
  @Column(name = "version")
  private int version;

  @Column
  private String nome;

  @Column
  private String sobrenome;

  @Column(length = 2000)
  private String bio;

  @Column
  private String twitter;
{% endhighlight %}

Defina o `JSF` como framework padrão para a camada de apresentação:

{% highlight bash %}
[Speaker.java]$  faces-setup --facesVersion 2.2
{% endhighlight %}

O JBoss Forge, possui uma funcionalidade chamada `scaffold`, que gera todo o arcabouço inicial de uma aplicação MVC, como Camada de Apresentação (telas), Controladores, Dao, entre outras. Para isso, execute:

{% highlight bash %}
[Speaker.java]$  scaffold-generate --targets org.judcon.brasil.model.Palestrante
***SUCCESS*** CDI has been installed.
***SUCCESS*** EJB has been installed.
***SUCCESS*** Servlet API has been installed.
***SUCCESS*** JAX-RS has been installed.
***SUCCESS*** Scaffold was generated successfully.
[Speaker.java]$  build
{% endhighlight %}

Toda estrutura básica de uma aplicação web foi criada e está pronta para uso:

![](/images/201510-devops-03.png)

## Continuous integration

Um desenvolvedor acaba de comitar uma mudança do sistema no repositório que contém a versão do código fonte, mas quando os outros desenvolvedores do projeto realizaram um checkout perceberam que o código não estava funcionando. Então enviaram um email relatando o problema, e depois de uma longa espera, a situação foi tratada e resolvida.

A prática de integração contínua tem como objetivo evitar situações como essa, tornando o processo de testes automatizado, garantindo assim qualidade no desenvolvimento das aplicações.  Com o auxilio de ferramentas como: Jenkins e JUnit, é possível ter um ambiente de integração continua dinâmico e flexível.

Um dos pontos positivos dessa prática, é  que o processo de configuração não é extenso, pode ser feito em poucos minutos sem uma preparação tão grande e não haverá a necessidade de se preocupar com uma nova configuração.  As configurações só serão alteradas caso a necessidade mude, como inclusão ou execlusão de etapas.

A desvantagem dessa prática, é que as funcionalidades serão disponibilizadas para o time de QA, somente se todos os testes forem realizados com sucesso, o que pode gerar alguns atritos em relação ao time de negócios.


### Publicando o Código Fonte no GitLab

Acesse novamente a URL http://gitlab e crie um novo grupo chamado devops:

![](/images/201510-devops-04.png)

![](/images/201510-devops-05.png)

![](/images/201510-devops-06.png)

Utilizando a funcionalidade `New Project`  crie um novo projeto chamado `conference` com a visibilidade`private`:

![](/images/201510-devops-07.png)

![](/images/201510-devops-08.png)

Clique no botão com o formato de uma `chave de boca` chamado `Admin Area`. Em seguida clique em `New User`:

![](/images/201510-devops-09.png)

Preencha as informações com os dados do desenvolvedor:

![](/images/201510-devops-10.png)

Observe que o GitLab iria enviar um Link para o email do usuário, mas como não foi configurado o SMTP, clique novamente em `User`, em seguida `Edit` no usuário para que a troca da senha seja realizada:

![](/images/201510-devops-11.png)

Navegue novamente até a página do projeto: [conference](http://gitlab/devops/conference/project_members  e clique em `Add Member`,  e `Add users to project`.  Não esqueça de configurar o `Project Access` com a role `Master`:

![](/images/201510-devops-12.png)

Faça `Logout` e  realize um novo login com o usuário `mmagnani` criado anteriormente.  Clique em `Profile Settings` → `SSH Keys` → `Add SSH Key` . Adicione a chave criada  `Add Key ` no tópico "Preparando a Máquina do Desenvolvedor":

![](/images/201510-devops-13.png)

A partir desse momento o desenvolvedor já está apto para publicar o projeto no GitLab.

Voltando para a máquina do desenvolvedor, navegue até o diretorio do projeto:

{% highlight bash %}
[mmagnani@localhost conference]$ pwd
/home/mmagnani/Development/PROJETOS/DevOps/conference
{% endhighlight %}

Inicialize o repositório do projeto Git:

{% highlight bash %}
[mmagnani@localhost conference]$ git init
Initialized empty Git repository in /home/mmagnani/Development/PROJETOS/DevOps/conference/.git/
{% endhighlight %}

Adicione todos os arquivos do projeto:

{% highlight bash %}
[mmagnani@localhost conference]$ git add .
{% endhighlight %}

Faça o primeiro commit:

{% highlight bash %}
[mmagnani@localhost conference]$ git commit -m "Primeiro Commit"
{% endhighlight %}

Adicione o repositório remoto que nesse caso está no GitLab:

{% highlight bash %}
[mmagnani@localhost conference]$ git remote add origin git@gitlab:devops/conference.git
{% endhighlight %}


Com o o código fonte para o repositório remoto:

{% highlight bash %}
[mmagnani@localhost conference]$ git push origin master
The authenticity of host 'gitlab (192.168.90.10)' can't be established.
ECDSA key fingerprint is 50:fd:30:b9:ed:67:07:d9:f6:d2:64:93:81:bd:db:1a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'gitlab,192.168.90.10' (ECDSA) to the list of known hosts.
Counting objects: 101, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (81/81), done.
Writing objects: 100% (101/101), 1.02 MiB | 0 bytes/s, done.
Total 101 (delta 5), reused 0 (delta 0)
To git@gitlab:devops/conference.git
 * [new branch]      master -> master
{% endhighlight %}

Navegue até o GitLab http://gitlab/devops/conference/activity. O código fonte da aplicação já está disponível:

![](/images/201510-devops-14.png)


###  Automatizando o processo de Build com Jenkins

Jenkins é um servidor de integração contínua open source. É utilizado em larga escala, para execução de  builds, testes e empacotamento das aplicações. O Jenkins possui recursos como: Groovy, API externa em JSON e XML,  plugins, além de outros recursos que facilitam tarefas de manutenção, migração, gerenciamento das aplicações e integrações com outros sistemas.

Crie um novo playbook chamado jenkins.yml e deixe a configuração como abaixo:

{% highlight yaml %}
- hosts: jenkins
  sudo: True
  user: vagrant
  tasks:
    - name: "Instala OpenJDK"
      yum: name=java-1.7.0-openjdk-devel state=latest

    - name: "Instala Git"
      yum: name=git state=latest

    - name: "Instala o wget e unzip"
      shell: sudo yum -y install wget unzip

    - name: "Download do Maven"
      get_url: url=http://mirror.nbtelecom.com.br/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.zip dest=/tmp/apache-maven-3.3.3-bin.zip

    - name: "Descompacta o Maven em /opt"
      unarchive: src=/tmp/apache-maven-3.3.3-bin.zip dest=/opt copy=no

    - name: "Download do repositório Jenkins"
      shell: sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo

    - name: "Cria um link simbólico para o Maven"
      shell: ln -s /opt/apache-maven-3.3.3 /opt/maven

    - name: "Importa a chave do repositório"
      shell: sudo rpm --import http://pkg.jenkins-ci.org/redhat-stable/jenkins-ci.org.key

    - name: "Instala Jenkins"
      shell: sudo yum -y install jenkins

    - name: "Habilita o jenkins"
      shell: sudo systemctl enable jenkins.service

    - name: "Inicia o jenkins"
      shell: sudo systemctl start jenkins.service

    - name: "Adiciona o IP do GitLab"
      lineinfile: 'dest=/etc/hosts line="192.168.90.10 gitlab"'

    - name: "Adiciona o IP do Nexus"
      lineinfile: 'dest=/etc/hosts line="192.168.90.30 nexus"'

    - name: "Chave SSH para Deploy no Gitlab"
      user: name=vagrant generate_ssh_key=yes
{% endhighlight %}

O arquivo VagrantFile também deve ser atualizado com a criação do novo servidor para o Jenkins e utilização do Playbook.

{% highlight ruby %}
VAGRANTFILE_API_VERSION = "2"
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "centos/7"

  config.vm.define :gitlab do |gitlab|
    gitlab.vm.hostname = "gitlab"
    gitlab.vm.network :private_network, :ip => "192.168.90.10"

    gitlab.vm.provision "ansible" do |ansible|
          ansible.playbook = "gitlab.yml"
          ansible.verbose = "vvv"
    end
 
    gitlab.vm.provider :libvirt do |domain|
      domain.memory = 1024
      domain.cpus = 2
    end

  end

  config.vm.define :jenkins do |jenkins|
    jenkins.vm.hostname = "jenkins"
    jenkins.vm.network :private_network, :ip => "192.168.90.20"

    jenkins.vm.provision "ansible" do |ansible|
          ansible.playbook = "jenkins.yml"
          ansible.verbose = "vvv"
    end
 
    jenkins.vm.provider :libvirt do |domain|
      domain.memory = 1024
      domain.cpus = 2
    end

  end

end

{% endhighlight %}

Para que o servidor e as configurações do Jenkins sejam criadas, basta executar o comando vagrant up:

{% highlight bash %}
[mmagnani@linux DevOps]$ vagrant up
{% endhighlight %}

Tudo ocorrendo conforme o esperado, navegue até a URL http://jenkins:8080 e a página inicial do Jenkins será exibida:

![](/images/201510-devops-15.png)


### WebHook x Pooling



### Instalando Plugins no Jenkins e Configurando GitLab WebHook

O Jenkins é um software bastante customizável e isso é alcançado através de plugins. Exitem plugins para as mais diversas finalidades como integração com GitLab, geração de relatório de testes, integraão com software APM (application performance management - Ex: Dynatrace), entre outros.

Como dito anteriormente, o Jenkins oferece integração nativa com Groovy. Para listar os plugins instalados atualmente, navegue até http://jenkins:8080/script e execute:

![](/images/201510-devops-16.png)

Observe o retorno:


{% highlight bash %}
Result

[Plugin:matrix-project, Plugin:javadoc, Plugin:credentials, Plugin:external-monitor-job, Plugin:mailer, Plugin:cvs, Plugin:antisamy-markup-formatter, Plugin:ant, Plugin:windows-slaves, Plugin:maven-plugin, Plugin:pam-auth, Plugin:translation, Plugin:ssh-credentials, Plugin:junit, Plugin:subversion, Plugin:matrix-auth, Plugin:ldap, Plugin:script-security, Plugin:ssh-slaves]
{% endhighlight %}

A instalação de plugin é feita de maneira simples. Acesse a opção `Manage Jenkins` localizada no menu lateral, em seguida Selecione `Manage Plugins`. Clique na aba Available. Selecione os seguintes plugins:

GitLab Logo Plugin  
Gitlab Hook Plugin
JaCoCo plugin

Em seguida, Install without restart e marque a opção Restart Jenkins when installation is complete and no jobs are running.

![](/images/201510-devops-17.png)

Acesse a opção `Manage Jenkins` localizada no menu lateral, clique em `Configure System`. Na opção `Maven`, deixe as configurações como abaixo e clique em Save.

![](/images/201510-devops-18.png)

Crie um novo JOB clicando em  `create new jobs` e preencha o nome e o tipo do projeto:

Nome:  Conference
Tipo: Maven project

![](/images/201510-devops-19.png)

Em `Source Code Management` selecione `Git` e preencha a opção `Repository URL` com a URL do projeto Conference: `git@gitlab:devops/conference.git` . Em `Branch Specifier (blank for 'any')` configure como `master`. A opção `Repository browser` pode permanecer como `Auto`. 

![](/images/201510-devops-20.png)

Clique em `Save`. A autenticação no repositório está falhando! Perceba que a autenticação do usuário jenkins no GitLab não existe e ainda não foi adicionada ao Jenkins.

Copie o valor da chave gerada na instalação do Jenkins:

{% highlight bash %}
[mmagnani@localhost DevOps]$ vagrant ssh jenkins
Last login: Wed Oct 21 13:54:17 2015 from 192.168.121.1
[vagrant@jenkins ~]$ cat /home/vagrant/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCvpeXzOxHO2LqPsAj+mvqd3BAaq7ep0PIBlT8iJQmEOOP2wp6G/hCkVZM3Kp42QYgmfIG8OUwfKTpazR3j/ftcB/LfNuDZJse1Xp0/B83B4he7MYHqJadwEgO8sRah0YUWK+yLNKRjDq+UEHuqqduOUFoeZNHxe7d9UDqTPG7Ei0Rjqo9gBiJuR3UobWPbqfXn8RGEhyF7/ydRtJsdghNxznnueuXIAtUpM+fu53xWF63LXjOtSpn+T0Nnj7YmvH4A2Wy+AC6Dnj/TWZ6dbPP3idOiej+0wmnWxzOTtdTO9cPzg79psH1aeJ0qq/JXHaPEKLU0aTVgZYdppihnZYrr ansible-generated on jenkins
{% endhighlight %}

Esse valor deve ser adicionado ao projeto no GitLab. Realize novamente login com usuário root . Clique no botão com o formato de uma `chave de boca` chamado `Admin Area`. Em seguida clique em `New User` e Preencha as informações com os dados do Jenkins:

![](/images/201510-devops-21.png)

Veja novamente que o GitLab iria enviar um link para o email do usuário,  mas como não foi configurado o SMTP altere a senha manualmente. Clique em `User` e em seguida `Edit` no usuário para que a  troca da senha seja realizada.

Agora navegue novamente até a página do projeto http://gitlab/devops/conference/project_members  e clique em `Add Member` e `Add users to project`.  Não esqueça de configurar o `Project Access` com a role `Master`

![](/images/201510-devops-22.png)

![](/images/201510-devops-23.png)

Faça `Logout` e  realize um novo login com o usuário jenkins.  Clique em `Profile Settings` → `SSH Keys` → `Add SSH Key` (o conteúdo da chave é  /home/vagrant/.ssh/id_rsa.pub) . Adicione a chave criada  `Add Key` gerada no passo anterior:

![](/images/201510-devops-24.png)

Retorne ao Jenkins e clique em Configure no JOB Conference: http://jenkins:8080/job/Conference/configure

Em Credentials clique em Add e preencha com as seguintes informações:

![](/images/201510-devops-25.png)

A Private Key se encontram em /home/vagrant/.ssh/id_rsa.

![](/images/201510-devops-26.png)

Clique em Save. A partir desse momento, o Jenkins já está apto para executar o primeiro Build da aplicação.

Implementando umas das práticas de DevOps, a integração continua deve ser executada automaticamente a cada mudança no repositório. Isso pode ser feito através de um mecanismo chamado Webhook. Webhooks permitem que sistemas externos recebam notificações de todos os eventos que ocorrem no sistema. Quando um evento acontece, o GitLab envia uma requisição HTTP POST para a URL configurada no webhook com as informações relativas ao evento. Ao receber a notificação, o Jenkins pode executar diversos procedimentos, com por exemplo o Build da aplicação.

Realize novamente login com usuário root . Clique no botão com o formato de uma `chave de boca` chamado `Admin Area`.   Clique no projeto Conference, em seguida `Edit → Web Hooks` e preencha com a URL do Jenkins habilitada para esse fim utilizando o Plugin Gitlab WebHook.

http://jenkins:8080/gitlab/build_now

![](/images/201510-devops-27.png)


Para verificar o funcionamento, retorne para a máquina do desenvolvedor e crie um arquivo chamado README no diretório raiz do projeto. Realize os procedimentos para commit e push:

{% highlight bash %}
[mmagnani@localhost conference]$ echo "Testando WebHook" >> README
[mmagnani@localhost conference]$ git add README
[mmagnani@localhost conference]$ git commit -m "Testando WebHook" README
[mmagnani@localhost conference]$ git push origin master
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 288 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To git@gitlab:devops/conference.git
   2afad3a..ef385a4  master -> master
{% endhighlight %}

Retorne ao Jenkins e veja que o Build foi realizado automaticamente:

![](/images/201510-devops-28.png)

No servidor do Jenkins em  /var/log/jenkins/jenkins.log, pode-se verificar o payload enviado pelo GitLab com o WebHook:

{% highlight bash %}
Oct 21, 2015 2:24:18 PM org.jruby.javasupport.JavaMethod invokeDirectWithExceptionHandling
INFO: matching projects:
   - Conference
Oct 21, 2015 2:24:18 PM org.jruby.javasupport.JavaMethod invokeDirectWithExceptionHandling
INFO: Conference scheduled for build
Oct 21, 2015 2:25:15 PM org.jruby.javasupport.JavaMethod invokeDirectWithExceptionHandling
INFO: gitlab web hook triggered for
   - repo url: git@gitlab:devops/conference.git
   - branch: master
   - with payload:
{
  "object_kind": "push",
  "before": "2afad3aa834d913ad949c0ca19be370db044c7ec",
  "after": "ef385a4b04fe1184bac9933448b44bd6c8b8e5c2",
  "ref": "refs/heads/master",
  "checkout_sha": "ef385a4b04fe1184bac9933448b44bd6c8b8e5c2",
  "message": null,
  "user_id": 4,
  "user_name": "Mauricio Magnani Jr",
  "user_email": "mmagnani@redhat.com",
  "project_id": 2,
  "repository": {
    "name": "conference",
    "url": "git@gitlab:devops/conference.git",
    "description": "Projeto destinado aos palestrantes do FISL16",
    "homepage": "http://gitlab/devops/conference",
    "git_http_url": "http://gitlab/devops/conference.git",
    "git_ssh_url": "git@gitlab:devops/conference.git",
    "visibility_level": 0
  },
  "commits": [
    {
      "id": "ef385a4b04fe1184bac9933448b44bd6c8b8e5c2",
      "message": "Testando WebHook\n",
      "timestamp": "2015-10-21T16:25:13-02:00",
      "url": "http://gitlab/devops/conference/commit/ef385a4b04fe1184bac9933448b44bd6c8b8e5c2",
      "author": {
        "name": "Mauricio Magnani Jr",
        "email": "mmagnani@redhat.com"
      }
    }
  ],
  "total_commits_count": 1
}
Oct 21, 2015 2:25:15 PM org.jruby.javasupport.JavaMethod invokeDirectWithExceptionHandling
INFO: matching projects:
   - Conference
Oct 21, 2015 2:25:15 PM org.jruby.javasupport.JavaMethod invokeDirectWithExceptionHandling
INFO: Conference scheduled for build
Oct 21, 2015 2:28:27 PM hudson.model.Run execute
{% endhighlight %}


Todo o processo foi realizado com sucesso utilizando o mecanismo de WebHook. 

Os arquivos utilizados até o momento estão no GitHub: https://github.com/msmagnanijr/devops-blog


[DevOps - Uma Abordagem Prática - Capítulo I](http://mmagnani.me/2015/10/21/devops-provisionamento-1)

[DevOps - Uma Abordagem Prática - Capítulo III](http://mmagnani.me/2015/10/25/devops-provisionamento-3)

[DevOps - Uma Abordagem Prática - Capítulo IV](http://mmagnani.me/2015/10/26/devops-provisionamento-4)