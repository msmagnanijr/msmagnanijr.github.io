---
layout: post
title: DevOps - Uma Abordagem Prática - Capítulo IV
modified: 2015-10-22
tags: [DevOps, Arquillian, WildFly, Ansible, Vagrant, Jenkins, Nexus, Pipeline]
comments: true
---

## Integração Contínua e Implantação contínua com Jenkins Build Pipeline

### JaCoCo: Melhorando a Cobertura dos Testes

Ferramentas que analisam a cobertura de código, tais como o Cobertura e JaCoCo são bastante úteis para ajudar a identificar trechos de código que não possuem testes. Vale ressaltar que 100% de cobertura de código, na maioria das vezes, não é necessário e nem factível. Cabe ao desenvolvedor avaliar a real necessidade.


Para automatizar a análise da cobertura de testes da aplicação conference, será utilizado plugin JaCoCo que foi instalado inicialmente no Jenkins.

![](/images/201510-devopsjacoco01.png)

Edite o JOB [Conference](http://jenkins:8080/job/Conference/configure) e na opção `Build` e `Goals and options`, adicione os parâmetros para o report do JaCoCo:

{% highlight bash %}
mvn jacoco:report test -Parquillian-wildfly-remote
{% endhighlight %}

![](/images/201510-devopsjacoco00.png)

adicione em `Add Post-build Actions` a opção `Record JaCoCo coverage report`. É necessário configurar três informações:

* Caminho do arquivo jacoco.exec.

  Deve ser preenchido com **/jacoco.exec.

* Caminho do diretório que contém os arquivos .class. 

  Deve ser preenchido com **/target/classes.

* Caminho do diretório de código fonte do projeto conference. 

  Deve ser preenchido com **/src/main/java.

![](/images/201510-devopsjacoco02.png)

Salve as alterações.

Na aplicação conference edite o `pom.xml` e adicione o plugin:

{% highlight xml %}
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.7.6-SNAPSHOT</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

Faça o `git commit` e `push` da alteração no `pom.xml`:

{% highlight bash %}
[mmagnani@localhost conference]$ git commit -m "Testes com JaCoCo " pom.xml
[master c94ddba] Testes com JaCoCo
 1 file changed, 147 insertions(+), 127 deletions(-)
 rewrite pom.xml (98%)
[mmagnani@localhost conference]$ git push origin master
{% endhighlight %}

O JaCoCo foi executado com sucesso no Build:

{% highlight bash %}
[JaCoCo plugin] Collecting JaCoCo coverage data...
[JaCoCo plugin] **/target/**.exec;**/target/classes;**/src/main/java; locations are configured
[JaCoCo plugin] Number of found exec files for pattern **/target/**.exec: 1
[JaCoCo plugin] Saving matched execfiles:  /var/lib/jenkins/jobs/Conference/workspace/target/jacoco.exec
[JaCoCo plugin] Saving matched class directories for class-pattern: **/target/classes:  /var/lib/jenkins/jobs/Conference/workspace/target/classes
[JaCoCo plugin] Saving matched source directories for source-pattern: **/src/main/java:  /var/lib/jenkins/jobs/Conference/workspace/src/main/java
[JaCoCo plugin] Loading inclusions files..
[JaCoCo plugin] inclusions: []
[JaCoCo plugin] exclusions: []
[JaCoCo plugin] Thresholds: JacocoHealthReportThresholds [minClass=0, maxClass=0, minMethod=0, maxMethod=0, minLine=0, maxLine=0, minBranch=0, maxBranch=0, minInstruction=0, maxInstruction=0, minComplexity=0, maxComplexity=0]
[JaCoCo plugin] Publishing the results..
[JaCoCo plugin] Loading packages..
[JaCoCo plugin] Done.
{% endhighlight %}

No jenkins os relatório estão disponíveis em [Code Coverage Trend](http://jenkins:8080/job/Conference/lastBuild/jacoco/) :

![](/images/201510-devopsjacoco03.png)

Como dito anteriormente o objetivo desse tutorial é demonstrar a parte operacional e quais ferramentas utilizar, sendo assim não será realizada a criação de um novo teste para melhorar a cobertura dó código mas nada impede que o leitor implemente por conta própria.


### Configurando WildFly Maven Plugin

O [WildFly Maven plugin](https://docs.jboss.org/wildfly/plugins/maven/latest/) é utilizado para deploy e undeploy de aplicações. Ele pode ser utilizado também para deploy de outros artefatos com driver JDBC. Também é possível executar comandos utilizando o JBoss CLI.

Esse plugin será utilizado para implementação da prática de Continuous Deployment (Implantação contínua) da aplicação conference.

Altere o `pom.xml` da aplicação confence e adicione o plugin do wildfly:

{% highlight xml %}
      <profile>
          <id>wildfly-remote</id>
          <build>     
          <plugins>    
            <plugin>
            <groupId>org.wildfly.plugins</groupId>
            <artifactId>wildfly-maven-plugin</artifactId>
            <version>1.0.2.Final</version>
            <configuration>
              <hostname>${wildfly-hostname}</hostname>
              <port>${wildfly-port}</port>
              <username>${wildfly-username}</username>
              <password>${wildfly-password}</password>
            </configuration>
          </plugin>
          </plugins>
          </build>
      </profile>
{% endhighlight %}

Altere também o arquivo `arquillian.xml` e deixe-o como abaixo:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<arquillian xmlns="http://jboss.org/schema/arquillian" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://jboss.org/schema/arquillian http://jboss.org/schema/arquillian/arquillian_1_0.xsd">
  <container qualifier="arquillian-wildfly-remote"  default="true">
        <configuration>
            <property name="managementAddress">{wildfly-hostname}</property>
            <property name="managementPort">${wildfly-port}</property>
            <property name="username">${wildfly-username}</property>
            <property name="password">${wildfly-password}</property>
        </configuration>
    </container>    
</arquillian>
{% endhighlight %}

Faça um pequeno teste para  verificar o funcionamento das alterações. Na máquina do desenvolvedor no projeto `conference` execute:

{% highlight bash %}
[mmagnani@localhost conference]$ mvn wildfly:deploy -Pwildfly-remote -Dwildfly-hostname=wildfly -Dwildfly-port=9990 -Dwildfly-username=devops -Dwildfly-password=devops@2015 -DskipTests=true
INFO: JBoss Remoting version 4.0.3.Final
Authenticating against security realm: ManagementRealm
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 11.353 s
[INFO] Finished at: 2015-10-25T19:15:40-02:00
[INFO] Final Memory: 15M/136M
[INFO] ------------------------------------------------------------------------

{% endhighlight %}

Esse procedimento realiza o deploy da aplicação `conference` no servdor remoto wildfly. Navegue até a URL [http://wildfly:8080/conference/](http://wildfly:8080/conference/) e veja se a aplicação realmente foi implantada:

 ![](/images/201510-mavenwildfly01.png)

Faça um teste com o JUnit e Arquillian:

{% highlight bash %}
[mmagnani@localhost conference]$ mvn jacoco:report test -Parquillian-wildfly-remote -Dwildfly-hostname=wildfly -Dwildfly-port=9990 -Dwildfly-username=devops -Dwildfly-password=devops@2015
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.conference.view.SpeakerBeanTest
Oct 25, 2015 7:20:57 PM org.xnio.Xnio <clinit>
INFO: XNIO version 3.2.0.Beta4
Oct 25, 2015 7:20:57 PM org.xnio.nio.NioXnio <clinit>
INFO: XNIO NIO Implementation Version 3.2.0.Beta4
Oct 25, 2015 7:20:57 PM org.jboss.remoting3.EndpointImpl <clinit>
INFO: JBoss Remoting version (unknown)
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 6.773 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 15.093 s
[INFO] Finished at: 2015-10-25T19:21:02-02:00
[INFO] Final Memory: 22M/295M
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

O teste também foi realizado com sucesso e já pode ser utilizado no Pipeline.

### Criando o Pipeline Inicial

Basicamente um pipeline na cultura DevOps é a automatização do processo de entrega de software, desde a alteração, testes, até sua implantação.

Mais informações sobre pipeline podem ser encontradas no livro [DevOps na prática: entrega de software confiável e automatizada](http://www.casadocodigo.com.br/products/livro-devops).

Atualmente a arquitetura proposta já possui dois passos distintos:

* Testes de integração com Arquillian;

* Build e implantação no WildFLy;

Para implementar o pipeline será utilizado o [Build Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin) do Jenkins. Para a instalação desse plugin acesse a opção `Manage Jenkins` localizada no menu lateral, em seguida Selecione `Manage Plugins`. Clique na aba Available. Selecione o  plugin:

Build Pipeline Plugin;

![](/images/201510-pipeline01.png)

 Crie um novo JOB chamado `Conference_Implantacao` baseado no JOB `Conference`:

 ![](/images/201510-pipeline01-1.png)

Em `Build` e `Goals and options` e adicione as configurações do WildFly remoto:

{% highlight bash %}
mvn wildfly:deploy -Pwildfly-remote -Dwildfly-hostname=wildfly -Dwildfly-port=9990 -Dwildfly-username=devops -Dwildfly-password=devops@2015 -DskipTests=true
{% endhighlight %}

 ![](/images/201510-pipeline01-2.png)

 Remova o `Post-build Actions` relacionado ao `JaCoCo` e clique em `Save`.

 Novamente crie um novo JOB chamado `Conference_Integracao` baseado no JOB `Conference`:

![](/images/201510-pipeline02.png)

Altere o `Build` e `Goals and options` e adicione as configurações do WildFly remoto:

{% highlight bash %}
mvn jacoco:report test -Parquillian-wildfly-remote -Dwildfly-hostname=wildfly -Dwildfly-port=9990 -Dwildfly-username=devops -Dwildfly-password=devops@2015
{% endhighlight %}

![](/images/201510-pipeline03.png)

Em `Post-build Actions` adicione `trigger parameterized build on other projects`. Deixe-o como abaixo:

![](/images/201510-pipeline04.png)

Clique em `Save`.

Essa ação diz ao Jenkins para disparar o JOB `Conference_Implantacao` se o Build estiver estável.

Remova o JOB `Conference`.

Clicando no símbolo de `+` do lado da palavra All crie uma nova View chamada `Conference_Pipeline` do tipo `Build Pipeline View`:

![](/images/201510-pipeline05.png)

![](/images/201510-pipeline06.png)

Na opção `Select Initial Job` selecione `Conference_Integracao`. Em `No Of Displayed Builds` coloque o valor `3`:

![](/images/201510-pipeline07.png)

Clique em `OK`. Agora que a  visualização do pipeline está criada, basta clicar em `Run` e observar a execução dos JOBs:

![](/images/201510-pipeline08.png)

![](/images/201510-pipeline09.png)

Veja que o JOB `Conference_Integracao` não foi executado com sucesso, sendo assim o JOB  `Conference_Implantacao` não foi disparado!

O que pode ter acontecido? Simples, as alterações do pom.xml e do arquillian.xml ainda não foram enviadas para o repositório remoto.

{% highlight bash %}
[mmagnani@localhost conference]$ git commit -m "Testando o Pipeline" .
[mmagnani@localhost conference]$ git push origin master
{% endhighlight %}

Como os projetos estão configurados com o mecanismo de WebHook o pipeline foi executado automaticamente ao enviar as alterações para o GitLab. Um novo erro ocorreu mas agora é nas configurações. Nos dois JOBs remova o parâmetro mvn:

![](/images/201510-pipeline10.png)

![](/images/201510-pipeline11.png)

Navegue novamente para o pipeline [http://jenkins:8080/view/Conference_Pipeline](http://jenkins:8080/view/Conference_Pipeline) e clique em 
`Run`:

![](/images/201510-pipeline12.png)

Depois de algumas pequenas correções o Pipeline foi executado com sucesso!

![](/images/201510-pipeline13.png)

Acesse a URL da aplicação [Conference](http://wildfly:8080/conference), que foi testada e implantada remotamente através de um pipeline de entrega:

![](/images/201510-pipeline14.png)

Durante os passos anteriores o foco foi no funcionamento e em nenhum momento foi implementando boas práticas de segurança em relação as senhas, ajustes nos repositórios para um novo release ou até mesmo a escalabilidade do Jenkins.

Lembre-se que esse é um assunto muito extenso e muita coisa ainda será refatorada e evoluída conforme a necessidade da arquitetura.

### Referências

[WildFly Maven Plugin](https://docs.jboss.org/wildfly/plugins/maven/latest/examples/deployment-example.html)

[JaCoCO](http://eclemma.org/jacoco/trunk/doc/maven.html)

[WildFly](http://wildfly.org)

[DevOps - Uma Abordagem Prática - Capítulo I](http://mmagnani.me/2015/10/21/devops-provisionamento-1)

[DevOps - Uma Abordagem Prática - Capítulo II](http://mmagnani.me/2015/10/22/devops-provisionamento-2)

[DevOps - Uma Abordagem Prática - Capítulo III](http://mmagnani.me/2015/10/25/devops-provisionamento-3)