---
layout: post
title: DevOps - Uma Abordagem Prática - Capítulo III
modified: 2015-10-22
tags: [DevOps, Arquillian, WildFly, Ansible, Vagrant, JUnit]
comments: true
---

##  Integração Contínua com Jenkins, JUnit, Arquillian e JaCoCo

No livro [Agile Desenvolvimento de software com entregas frequentes e foco no valor de negócio](http://www.casadocodigo.com.br/products/livro-agile), o autor André Faria Gomes descreve uma ótima definição sobre os objetivos da integração contínua:

"Manter o código sempre integrado e pronto para ser entregue é um grande desafio, e essa é uma das finalidades da integração contínua. A ideia da integração contínua é que todos os desenvolvedores realizem integração, isso é, sincronizem o código fonte produzido com o sistema de controle de versão, a maior quantidade de vezes possível ao longo do dia".

É de suma importância que a cada evento no repositório de código fonte a adição de novas funcionalidades sejam testadas de forma adequada. Isso pode ser alcançando utilizando testes unitários e testes de integração.

* Testes unitários são responsáveis por testar a menor unidade de uma aplicação, como por exemplo, uma classe ou um método específico. 

* Teste de integração tem por objetivo testar a integração de várias classes (componentes) de uma aplicação, visando simular um comportamento.

Existem algumas maneiras de separar os dois tipos de testes e nessa [thread do stackoverflow](http://stackoverflow.com/questions/2606572/junit-splitting-integration-test-and-unit-tests) alguns exemplos são discutidos. Como o objetivo desse tutorial é focar mais na parte operacional do ambiente, será criado um teste simples para que os plugins e o ambiente de integração possa ser simulado.

Boas referências:

[Testes automatizados de software: Um guia prático](http://www.casadocodigo.com.br/products/livro-testes-de-software)

[Test-Driven Development: Teste e Design no Mundo Real](http://www.casadocodigo.com.br/products/livro-tdd)

### Criando um novo caso de Teste com JUnit e Arquillian

Arquillian é um framework que permite aos desenvolvedores escrever e executar testes de integração de forma simples. O Arquillian utiliza JUnit ou TestNG para executar casos de teste em um container Java, como por exemplo o WildFly.

Ao iniciar um novo testes utilizando o Arquillian tenha em conta que eles espera encontrar os seguintes pré-requisitos:

* Pelo menos um método anotado com @Test quando JUnit for utilizado;

* A classe de testes deve utilizar a anotação @RunWith(Arquillian.class);

* Um método que retorne uma classe com os pacote e as classes que serão instaladas no container. Esse método deve utilizar a anotação @Deployment.

* Por último, um container onde será instalada a aplicação;

No JBoss Developer Studio, navegue até `Windows → Show View → Forge Console` e inicie o Forge utilizando o botão start. Instale o plugin
[Arquillian Forge 2 Addon](https://github.com/forge/addon-arquillian) utilizando o comando abaixo:

{% highlight xml %}
addon-install-from-git --url https://github.com/forge/addon-arquillian.git --coordinate org.arquillian.forge:arquillian-addon
{% endhighlight %}

![](/images/201510-testeunitario-01.png)

Execute o setup do Arquillian utilizando o JUnit e WildFly:

{% highlight xml %}
arquillian-setup --testFramework junit --containerAdapter wildfly-remote
{% endhighlight %}

![](/images/201510-testeunitario-02.png)

Configure um novo teste para o Speaker:

{% highlight xml %}
arquillian-create-test --targets org.conference.view.SpeakerBean  --archiveType JAR
{% endhighlight %}

![](/images/201510-testeunitario-03.png)

A classe de teste criada se chama SpeakerBeanTest. Deixe-a como abaixo:

{% highlight java %}
package org.conference.view;

package org.conference.view;

import javax.inject.Inject;

import org.conference.model.Speaker;
import org.jboss.arquillian.container.test.api.Deployment;
import org.jboss.arquillian.junit.Arquillian;
import org.jboss.shrinkwrap.api.Archive;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.asset.EmptyAsset;
import org.jboss.shrinkwrap.api.spec.WebArchive;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(Arquillian.class)
public class SpeakerBeanTest {

	@Inject
	private SpeakerBean speakerBean;
	
	
	private Speaker speaker = new Speaker();

	@Deployment
    public static Archive<?> createDeployment() {
	
		
        return ShrinkWrap.create(WebArchive.class, "conference-1.0.0-SNAPSHOT.war")
		        .addClass(SpeakerBean.class).addClass(Speaker.class)
		        .addAsResource("META-INF/persistence.xml")
				.addAsManifestResource(EmptyAsset.INSTANCE, "beans.xml");
	}

	@Test
	public void should_be_deployed() {
	 
		speaker.setFirstname("Mauricio");
		speaker.setSurname("Magnani");
		speaker.setBio("Consultor de Tecnologia");
		speaker.setTwitter("msmagnanijr");
		
		speakerBean.setSpeaker(speaker);
		
		Assert.assertNotNull(speakerBean);
		Assert.assertEquals("Paulo", speakerBean.getSpeaker().getFirstname());
	}
}
{% endhighlight %}

Como dito anteriormente, para o que teste seja executado é necessário um container Java. Nessa arquitetura será utilizado o WildFly 8.2.1. Para a instalação do WildFly crie um novo playbook chamado `wildfly.yml` e deixe-o como abaixo:

{% highlight xml %}
- hosts: wildfly
  sudo: True
  user: vagrant
  tasks:
    - name: "Instala OpenJDK"
      yum: name=java-1.8.0-openjdk-devel state=latest

    - name: "Instala o wget e unzip"
      shell: sudo yum -y install wget unzip

    - name: "Instala o OpenSSL"
      yum: name=openssl state=latest

    - name: "Download do WildFly"
      get_url: url=http://download.jboss.org/wildfly/8.2.1.Final/wildfly-8.2.1.Final.zip dest=/tmp/wildfly-8.2.1.Final.zip

    - name: "Descompacta o WildFly em /opt"
      unarchive: src=/tmp/wildfly-8.2.1.Final.zip dest=/opt copy=no

    - name: "Cria link simbolico para o WildFly"
      shell: sudo ln -s /opt/wildfly-* /opt/wildfly

    - name: "Copia o arquivo wildfly.service para criar o serviço"
      copy: src=wildfly.service  dest=/etc/systemd/system  

    - name: "Permissão do serviço"
      shell: sudo chmod 644 /etc/systemd/system/wildfly.service

    - name: "Usuario WildFly"
      shell: sudo useradd -p `openssl passwd -1 wildfly` wildfly

    - name: "Adiciona ao Grupo wildfly"
      shell: sudo usermod -aG wildfly wildfly

    - name: "Permissão do diretório"
      shell: sudo chown -R wildfly:wildfly /opt/wildfly*

    - name: "Configura o usuario do wildfly como sudo"
      lineinfile: 'dest=/etc/sudoers  line="wildfly ALL=(ALL) NOPASSWD:ALL" state=present  validate="visudo -cf %s"'

    - name: "Habilita o servico do WildFly"
      shell: sudo  systemctl enable wildfly.service      

    - name: "Inicia o servico do WildFly"
      shell: sudo  systemctl start wildfly.service
{% endhighlight %}

 No diretório onde estão todas as configurações, crie um novo arquivo chamado `wildfly.service` que será utilizado pelo playbook `wildfly.yml`:

{% highlight bash %}
[Unit]
Description=WildFly 8.2.1
After=syslog.target network.target

[Service]
Type=simple
ExecStart=/opt/wildfly/bin/standalone.sh

[Install]
WantedBy=multi-user.target
{% endhighlight %}

Para finalizar altere o arquivo `Vagrantfile` adicionando o playbook `wildfly.yml` e descrevendo a criação do novo servidor:

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

  config.vm.define :wildfly do |wildfly|
    wildfly.vm.hostname = "wildfly"
    wildfly.vm.network :private_network, :ip => "192.168.90.50"

    wildfly.vm.provision "ansible" do |ansible|
          ansible.playbook = "wildfly.yml"
          ansible.verbose = "vvv"
    end
 
    wildfly.vm.provider :libvirt do |domain|
      domain.memory = 1024
      domain.cpus = 2
    end

  end

end
{% endhighlight %}

Na máquina do desenvolvedor, altere o arquivo `/etc/hosts` e adicione o IP e nome do WildFly:

{% highlight bash %}
[mmagnani@localhost DevOps]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.90.10 gitlab
192.168.90.20 jenkins
192.168.90.50 wildfly
{% endhighlight %}

Para iniciar a configuração com o playbook, utilize o comando `vagrant up`:

{% highlight bash %}
[mmagnani@linux DevOps]$ vagrant up
{% endhighlight %} 


Terminada a instalação, navegue até a URL [http://wildfly:8080](http://wildfly:8080), observe que o WildFly ainda não está acessível.

Acesse o servidor wildfly utilizando SSH com o comando `vagrant ssh wildfly`:

{% highlight xml %}
[mmagnani@localhost DevOps]$ vagrant ssh wildfly
{% endhighlight %}

Edite o arquivo `/opt/wildfly/bin/standalone.conf` e adicione as configurações de binding para a variável `JAVA_OPTS`:

{% highlight bash %}
if [ "x$JAVA_OPTS" = "x" ]; then
   JAVA_OPTS="-Xms64m -Xmx512m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true"
   JAVA_OPTS="$JAVA_OPTS -Djboss.modules.system.pkgs=$JBOSS_MODULES_SYSTEM_PKGS -Djava.awt.headless=true"
   JAVA_OPTS="$JAVA_OPTS -Djboss.bind.address.management=192.168.90.50 -Djboss.bind.address=192.168.90.50"
{% endhighlight %}

Reinicie o WildFly:

{% highlight bash %}
[vagrant@wildfly bin]$ sudo  systemctl restart wildfly.service
{% endhighlight %}

Acesse novamente a URL [http://wildfly:8080](http://wildfly:8080) e a página inicial do WildFly será exibida:

![](/images/201510-testeunitario-04.png)

Ainda no servidor do WildFly crie um usuário de gerenciamento para ser utilizados nos testes de integração e deploy. Adione um usuário com a `Role ( ManagementRealm )` de Gerenciamento como abaixo:

{% highlight bash %}
[vagrant@wildfly bin]$ pwd
/opt/wildfly/bin
[vagrant@wildfly bin]$ sudo ./add-user.sh 

What type of user do you wish to add? 
 a) Management User (mgmt-users.properties) 
 b) Application User (application-users.properties)
(a): a

Enter the details of the new user to add.
Using realm 'ManagementRealm' as discovered from the existing property files.
Username : devops
Password recommendations are listed below. To modify these restrictions edit the add-user.properties configuration file.
 - The password should not be one of the following restricted values {root, admin, administrator}
 - The password should contain at least 8 characters, 1 alphabetic character(s), 1 digit(s), 1 non-alphanumeric symbol(s)
 - The password should be different from the username
Password : devops@2015
Re-enter Password : devops@2015
What groups do you want this user to belong to? (Please enter a comma separated list, or leave blank for none)[  ]: 
About to add user 'devops' for realm 'ManagementRealm'
Is this correct yes/no? yes
Added user 'devops' to file '/opt/wildfly-8.2.1.Final/standalone/configuration/mgmt-users.properties'
Added user 'devops' to file '/opt/wildfly-8.2.1.Final/domain/configuration/mgmt-users.properties'
Added user 'devops' with groups  to file '/opt/wildfly-8.2.1.Final/standalone/configuration/mgmt-groups.properties'
Added user 'devops' with groups  to file '/opt/wildfly-8.2.1.Final/domain/configuration/mgmt-groups.properties'
Is this new user going to be used for one AS process to connect to another AS process? 
e.g. for a slave
{% endhighlight %}

Navegue até a URL [http://wildfly:9990](http://wildfly:9990) e faça login com o usuário `devops` no painel de gerenciamento do WildFly:

![](/images/201510-testeunitario-05.png)

A instalação e configuração do WildFly está finalizada.

Altere a aplicação conference. Edite o arquivo `arquillian.xml` e adicione as configurações do WildFly remoto:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<arquillian xmlns="http://jboss.org/schema/arquillian" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://jboss.org/schema/arquillian http://jboss.org/schema/arquillian/arquillian_1_0.xsd">
  <container qualifier="arquillian-wildfly-remote">
        <configuration>
            <property name="managementAddress">wildfly</property>
            <property name="managementPort">9990</property>
            <property name="username">devops</property>
            <property name="password">devops@2015</property>
        </configuration>
    </container>    
</arquillian>
{% endhighlight %}

No JOB [Conference](http://jenkins:8080/job/Conference/configure) altere o parâmetro `Goals and options` no `Build`:

![](/images/201510-testeunitario-10.png)


Na raíz do projeto `conference` crie um arquivo chamado `.gitignore`,  isso evita que alguns arquivos desnecessários sejam enviados para o repositório:

{% highlight bash %}
.classpath
.project
.settings/
target/
{% endhighlight %}

Faça o `git commit` e `push` de todas as alterações:

{% highlight xml %}
[mmagnani@localhost conference]$ pwd
/home/mmagnani/Development/PROJETOS/DevOps/conference
[mmagnani@localhost conference]$ git add .
[mmagnani@localhost conference]$ git commit -m "Testes com Arquillian e JUnit"
[mmagnani@localhost conference]$ git push origin master
{% endhighlight %}

Navegue até o JOB Conference no Jenkins e veja que o teste falhou:

{% highlight bash %}
 T E S T S
-------------------------------------------------------
Running org.conference.view.SpeakerBeanTest
Oct 24, 2015 10:24:42 PM org.xnio.Xnio <clinit>
INFO: XNIO version 3.2.0.Beta4
Oct 24, 2015 10:24:43 PM org.xnio.nio.NioXnio <clinit>
INFO: XNIO NIO Implementation Version 3.2.0.Beta4
Oct 24, 2015 10:24:43 PM org.jboss.remoting3.EndpointImpl <clinit>
INFO: JBoss Remoting version (unknown)
Tests run: 1, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 7.349 sec <<< FAILURE!
should_be_deployed(org.conference.view.SpeakerBeanTest)  Time elapsed: 0.494 sec  <<< FAILURE!
org.junit.ComparisonFailure: expected:<[Paul]o> but was:<[Maurici]o>
	at org.junit.Assert.assertEquals(Assert.java:115)
	at org.junit.Assert.assertEquals(Assert.java:144)
	at org.conference.view.SpeakerBeanTest.should_be_deployed(SpeakerBeanTest.java:46)


Results :

Failed tests: 
  SpeakerBeanTest.should_be_deployed:46 expected:<[Paul]o> but was:<[Maurici]o>

Tests run: 1, Failures: 1, Errors: 0, Skipped: 0

[ERROR] There are test failures.
{% endhighlight %}

O erro é claro: Diz que esperava o nome `"Paulo"` mas veio `"Mauricio"` por isso ocorreu a falha:

{% highlight bash %}
expected:<[Paul]o> but was:<[Maurici]o>
{% endhighlight %}

Altere a classe `SpeakerBeanTest` e agora adicine o Speaker `"Paulo"`:

{% highlight java %}
package org.conference.view;

import javax.inject.Inject;

import org.conference.model.Speaker;
import org.jboss.arquillian.container.test.api.Deployment;
import org.jboss.arquillian.junit.Arquillian;
import org.jboss.shrinkwrap.api.Archive;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.asset.EmptyAsset;
import org.jboss.shrinkwrap.api.spec.WebArchive;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(Arquillian.class)
public class SpeakerBeanTest {

	@Inject
	private SpeakerBean speakerBean;
	
	
	private Speaker speaker = new Speaker();

	@Deployment
    public static Archive<?> createDeployment() {
	
		
        return ShrinkWrap.create(WebArchive.class, "conference-1.0.0-SNAPSHOT.war")
		        .addClass(SpeakerBean.class).addClass(Speaker.class)
		        .addAsResource("META-INF/persistence.xml")
				.addAsManifestResource(EmptyAsset.INSTANCE, "beans.xml");
	}

	@Test
	public void should_be_deployed() {
	 
		speaker.setFirstname("Paulo");
		speaker.setSurname("Junior");
		speaker.setBio("Engenheiro");
		speaker.setTwitter("paulo2015");
		
		speakerBean.setSpeaker(speaker);
		
		Assert.assertNotNull(speakerBean);
		Assert.assertEquals("Paulo", speakerBean.getSpeaker().getFirstname());
	}
}
{% endhighlight %}

Faça commit e push dessa alteração:

{% highlight bash %}
[mmagnani@localhost conference]$ git commit -m "Novo teste com JUnit e Arquillian" src/test/java/org/conference/view/SpeakerBeanTest.java
[mmagnani@localhost conference]$ git push origin master
{% endhighlight %}

No JOB Conference no Jenkins observe que o teste foi executado com sucesso!

{% highlight bash %}
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.conference.view.SpeakerBeanTest
Oct 24, 2015 10:30:39 PM org.xnio.Xnio <clinit>
INFO: XNIO version 3.2.0.Beta4
Oct 24, 2015 10:30:39 PM org.xnio.nio.NioXnio <clinit>
INFO: XNIO NIO Implementation Version 3.2.0.Beta4
Oct 24, 2015 10:30:39 PM org.jboss.remoting3.EndpointImpl <clinit>
INFO: JBoss Remoting version (unknown)
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.283 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
{% endhighlight %}

Perceba que o Jenkins habilitou o gráfico `Test Result Trend`:

![](/images/201510-testeunitario-06.png)

A prática de integração contínua é de grande importância para alcançar excelência no desenvolvimento das aplicações. No próximo capítulo o assunto abordado será: JaCoCo: Melhorando a Cobertura dos Testes.

##  Centralizando Repositórios com Nexus

Em aplicações modernas construídas na plataforma Java cada vez mais gerenciadores de dependências são utilizados. Maven, Ivy e Gradle são alguns exemplos cada qual com a sua finalidade, vantagens e desvantagens.

Em relação ao Maven pode-se citar as seguintes vantagens:

* Padronização do layout de diretórios da aplicações;
* Gerenciamento de dependências;
* Padronização do build lifecycle;
* Build multi-projeto;

Uma das tarefas do Maven é o gerenciamento das dependências de uma aplicação. Este gerenciamento consiste na descrição de quais bibliotecas a aplicação utiliza. Essa descrição encontra-se no pom.xml da aplicação:

Exemplo:

{% highlight xml %}
<dependencies>
 ...
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>

 ...	
{% endhighlight %}


Por default o Maven busca os artefatos (bibliotecas) diretamente da internet em repositórios remotos. Essa abordagem traz algumas desvantagens na utilização em times de desenvolvimento, como por exemplo:

* Alteração da polítita de segurança para permitir downloads de arquivos `.JAR`:

Um time com mais de 40 desenvolvedores com exceções no proxy serial algo muito trabalhoso. 

* Aumento do consumo de banda de Internet:

Imagine os mesmos 40 desenvolvedores baixando as mesmas 200 bibliotecas, isso iria gerar um alto tráfego na rede e alto consumo de banda. 

* Bibliotecas internas (proprietárias) não podem ser encontradas automaticamente pois as dependências estão sendo buscadas na internet:

O time de desenvolvimento criou uma biblioteca chamada utils.jar que pode ser reaproveitada por todo o time mas para disponibilizá-la é necessário compartilhar um diretório na rede, hospedar em um servidor ou via FTP. Novamete uma tarefa trivial se tornou muito trabalhosa. 

O Nexus entra como solução para todas essas desvantagens. Ele é considerado uma ferramenta para repositório de artefatos, sendo que neste repositório, é realizado o upload das bibliotecas (JAR), deixando-as disponíveis para o download pelo Maven no momento do build da aplicação.

Para instalar o nexus crie um novo playbook no mesmo diretório dos anteriores chamado nexus.yml e adicione as seguintes informações:

{% highlight yaml %}
- hosts: nexus
  sudo: True
  user: vagrant
  tasks:
    - name: "Instala OpenJDK"
      yum: name=java-1.7.0-openjdk-devel state=latest

    - name: "Instala o OpenSSL"
      yum: name=openssl state=latest

    - name: "Instala o wget e unzip"
      shell: sudo yum -y install wget unzip

    - name: "Download do Nexus"
      get_url: url=http://www.sonatype.org/downloads/nexus-latest-bundle.zip dest=/tmp/nexus.zip

    - name: "Descompacta o Nexus em /opt"
      unarchive: src=/tmp/nexus.zip dest=/opt copy=no

    - name: "Cria link simbolico para o Nexus"
      shell: sudo ln -s /opt/nexus-* /opt/nexus

    - name: "Copia o arquivo nexus.service para criar o serviço"
      copy: src=nexus.service  dest=/etc/systemd/system  

    - name: "Permissão do serviço"
      shell: sudo chmod 644 /etc/systemd/system/nexus.service

    - name: "Configura usuario de serviço no Nexus"
      lineinfile: 'dest=/opt/nexus/bin/nexus line="RUN_AS_USER=nexus" insertafter="^#RUN_AS_USER=" state=present'

    - name: "Usuario Nexus"
      shell: sudo useradd -p `openssl passwd -1 nexus` nexus

    - name: "Adiciona ao Grupo Nexus"
      shell: sudo usermod -aG nexus nexus

    - name: "Permissão do diretório"
      shell: sudo chown -R nexus:nexus /opt/nexus* /opt/sonatype-work*

    - name: "Configura o usuario do Nexus como sudo"
      lineinfile: 'dest=/etc/sudoers  line="nexus ALL=(ALL) NOPASSWD:ALL" state=present  validate="visudo -cf %s"'

    - name: "Habilita o serviço do Nexus"
      shell: sudo  systemctl enable nexus.service      

    - name: "Inicia o serviço do Nexus"
      shell: sudo  systemctl start nexus.service
{% endhighlight %}

Atualize também o arquivo Vagrantfile adicionando o playbook nexus.yml e descrevendo a criação do novo servidor:

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

  config.vm.define :wildfly do |wildfly|
    wildfly.vm.hostname = "wildfly"
    wildfly.vm.network :private_network, :ip => "192.168.90.50"

    wildfly.vm.provision "ansible" do |ansible|
          ansible.playbook = "wildfly.yml"
          ansible.verbose = "vvv"
    end
 
    wildfly.vm.provider :libvirt do |domain|
      domain.memory = 1024
      domain.cpus = 2
    end

  end

  config.vm.define :nexus do |nexus|
    nexus.vm.hostname = "nexus"
    nexus.vm.network :private_network, :ip => "192.168.90.30"

    nexus.vm.provision "ansible" do |ansible|
          ansible.playbook = "nexus.yml"
          ansible.verbose = "vvv"
    end
 
    nexus.vm.provider :libvirt do |domain|
      domain.memory = 1024
      domain.cpus = 2
    end

  end

end
{% endhighlight %}

Na máquina do desenvolvedor, especificamente no arquivo `/etc/hosts` adicione o IP e nome do nexus para facilitar as configurações:

{% highlight bash %}
[mmagnani@localhost DevOps]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.90.10 gitlab
192.168.90.20 jenkins
192.168.90.50 wildfly
192.168.90.30 nexus
{% endhighlight %}

Ainda no diretório onde estão todas as configurações, crie um novo arquivo chamado nexus.service que será utilizado pelo playbook nexus.yml. Deixe-o como abaixo:

{% highlight bash %}
[Unit]
Description=Nexus Repository
After=syslog.target network.target

[Service]
Type=simple
ExecStart=/opt/nexus/bin/nexus start

[Install]
WantedBy=multi-user.target
{% endhighlight %}

Para iniciar a configuração com o playbook criado, utilize o comando `vagrant up`:

{% highlight bash %}
[mmagnani@linux DevOps]$ vagrant up
{% endhighlight %} 

Terminada a instalação com sucesso, navegue até a URL [http://nexus:8081/nexus](http://nexus:8081/nexus) e faça o login com o usuário e senha padrão:

Username: admin
Password: admin123

![](/images/201510-nexus-01.png)


A instalação do Nexus está finalizada!

### Conhecendo um Pouco Mais

O Nexus vem pré-configurado com alguns repositórios padrões. Os mais importantes são 3rd Party, Releases, e Snapshots:

* 3rd Party - Serve para armazenar bibliotecas terceiras (não produzidas pela equipe de desenvolvimento), como as do JBoss, Apache, JUnit, entre outros. 

* Releases -  Armazena as bibliotecas produzidas internamente pela equipe, porém apenas versões fechadas e homologadas.

* Snapshots -  Por fim, o repositório Snapshots, assim como o Releases, também armazenam bibliotecas internas, porém somente aquelas que estão em fase de desenvolvimento. 

![](/images/201510-nexus-02.png)

O Nexus possui também três categorias distintas de repositórios:

* Hosted -  São repositórios responsáveis pelos arquivos físicos das bibliotecas no Nexus. 

* Proxy - Responsável apenas por fazer referência a algum repositório público existente. Quando alguém requisita uma biblioteca para um repositório deste tipo, o Nexus apenas faz o link com repositório que é realmente responsável pela biblioteca. 


* Virtual - Repositórios virtuais, utilizados para manter compatibilidade entre as versões 1 e 2 do Maven. 

* Group - Serve para agrupar um conjunto de repositórios (hosted, virtual ou proxy). Deste modo, pode-se, por exemplo, criar um grupo chamado public, e colocar neste grupo apenas os repositórios onde é permitido o acesso aos desenvolvedores, criando assim uma política de acesso às bibliotecas.

![](/images/201510-nexus-03.png)

### Cache dos Artefatos

Seria muito custoso se o Maven, durante todo build de uma aplicação, fizesse o download de todas as dependências necessárias.  Para amenizar esse trabalho, o Maven sempre busca a dependência, num primeiro momento, em um cache local, que é preenchido toda vez que o download do artefato é realizado pela primeira vez no Nexus. Este cache fica, por padrão, no diretório .m2 do home do usuário, onde o Maven está instalado:

![](/images/201510-nexus-04.png)

### O Arquivo settings.xml

Por padrão, as dependências internas do Maven são sempre buscadas no repositório central, ao invés de serem referenciadas no Nexus local.
Veja que no build realizado até o momento as dependências foram buscadas na internet:

![](/images/201510-nexus-05.png)

Para alterar esse comportamento fazendo com que o Nexus tenha a responsabilidade de consultar o repositório central caso necessário, realize um ajuste ou crie o arquivo settings.xml onde o maven está instalado, ou seja na maquina do desenvolvedor e no servidor Jenkins: 

Obs: O usuário deployment é default do Nexus e pode ser utilizado para baixar ou publicar artefatos.

{% highlight xml %}
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <servers>
    <server>
      <id>deployment</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
  </servers>

  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://nexus:8081/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
</settings>
{% endhighlight %}

Faça login no servidor do Jenkins utilizando o comando `vagrant ssh jenkins`:

{% highlight bash %}
[mmagnani@localhost DevOps]$ vagrant ssh jenkins
{% endhighlight %}

No servidor navegue até o diretório  `/var/lib/jenkins/.m2/` e crie o arquivo settings.xml com o conteúdo acima.  Em seguida altere as permissões do arquivo para o usuário e grupo jenkins:


{% highlight bash %}
[vagrant@jenkins .m2]$ ls
repository  settings.xml
[vagrant@jenkins .m2]$ sudo chown jenkins:jenkins settings.xml 
[vagrant@jenkins .m2]$ ll
total 8
drwxr-xr-x. 14 jenkins jenkins 4096 Oct 24 08:45 repository
-rw-r--r--.  1 jenkins jenkins  565 Oct 24 09:24 settings.xml
{% endhighlight %}

Apague o diretório `repository`:

{% highlight bash %}
[vagrant@jenkins .m2]$ sudo rm -Rf repository
{% endhighlight %}

Navegue até o Jenkins no JOB Conference [http://jenkins:8080/job/Conference](http://jenkins:8080/job/Conference) e clique na opção `Build Now`. Depois clique na opção `Console`. O Jenkins já está utilizando o nexus para resolver as depêndecias da aplicação Conference:

 ![](/images/201510-nexus-06.png)

 O diretório `repository` no servidor do Jenkins será recriado automaticamente.

 A instalação do Nexus está finalizada.

### Referências

[JBoss Forge](http://forge.jboss.org/addon/org.arquillian.forge:arquillian-addon)

[Arquillian](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/RUN_ARQUILLIAN_TESTS.md)

[WildFly](http://wildfly.org)