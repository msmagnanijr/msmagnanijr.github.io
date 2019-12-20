---
layout: post
title: DevOps - Uma Abordagem Prática - Capítulo I
modified: 2016-01-11
tags: [DevOps, Vagrant, Virtual Box, Ansible, CI, CD]
comments: true
---

##  Iniciando com  DevOps

Para muitas organizações, a habilidade de inovar a passos rápidos respondendo às condições do mercado e ao feedback do cliente é um elemento chave para o sucesso. Empresas como Google, Facebook, SalesForce e Twitter, atualizam suas aplicações dezenas de vezes ao dia e tentam diminuir cada vez mais o tempo de disponibilização de novas funcionalidades, atualizações ou otimizações por meio de automação de tarefas, testes integrados, etc. Isso tem trazido grandes ganhos a essas corporações e ao mesmo tempo diminuido o risco de bugs, pois as atualizações são realizadas em "pacotes menores" tornando assim mais fácil a identificação de eventuais problemas. Essa cultura de automatização, integração contínua, teste em larga escala, tem sido chamada de DevOps. Mas o que realmente é DevOps?

DevOps não tem uma definição formal, porém sua filosofia não é novidade, pois muitos de seus princípios nasceram do movimento da metodologia ágil. DevOps pode ser descrito como um movimento de profissionais que pregam o trabalho com relacionamento colaborativo entre o time de desenvolvimento `(Dev)` e o time de operações `(Ops)`, resultando em um rápido fluxo de trabalho planejado, mantendo simultaneamente a confiabilidade, estabilidade, resiliência e segurança em um ambiente de produção. Os princípios e práticas de DevOps confirmam que a alta performance de TI é fortemente relacionada à performance do negócio, auxiliando a aumentar a produtividade e rentabilidade no mercado. Promovendo a integração e colaboração entre as áreas de desenvolvimento, operação e negócio.


### Desmistificando Alguns Termos

Ao iniciar a adoção e implementação de DevOps, será comum a utilização de alguns `termos/siglas` como `Continuous Integration, Continuous Delivery e Continuous Deployment, entre outras`. Qual a diferença entre elas?

`Continuous Integration` – Prática de integração ( merge brench / trunk ) e testes de um novo código com a base de código existente onde a estabilidade e confiabilidade desse novo código são garantidos através de testes automatizados.

`Continuous Inspection` – Milhares de funcionalidades e linhas de códigos são desenvolvidas todos os dias. Desenvolvedores estão a todo momento realizando algum tipo de manutenção ou refactoring. Mas como podemos verificar se o código está bem escrito ou com uma boa arquitetura? Se segue os padrões de mercado? Ou os padrões definidos pelo arquiteto do projeto? Se existe algum gargalo/problema conhecido nos códigos.desenvolvidos? Quanto tempo um analista gastaria para executar esses cenários? Continuous Inspection aumenta a visibilidade da qualidade de código para todas as partes interessadas e tornando-se parte integrante do ciclo de desenvolvimento de sofware.

`Continuous Testing` – É comum empresas realizarem testes de maneira manual, utilizando uma área de qualidade onde analistas de testes seguem determinados scritps baseado nos cenários desenvolvidos.Imagine uma aplicação de cadastros, quantas funcionalidades devem ser testadas nesse simples cenário? . É necessário mais agilidade nesse processo. "Continuous Testing é o processo de execução de testes automatizados como parte do pipeline de entrega de software para obter feedback imediato sobre os riscos de negócios associados a entrada de novas atualizações do sitema no ambiente de produção" Wikipedia.

`Continuous Deployment` – Está relacionado com a automatização da publicação de artefatos nos ambientes de produção, que passaram com sucesso pela fase de Continuous Integration, Continuous Inspection e Continuous Testing, . Essa automatização segue um padrão conhecido como `pipeline de entrega`.

`Continuous Monitoring` – Está relacionado ao monitoramento de infra-estrutura automática e contínua com o intuito de verificar o desempenho e a disponibilidade de aplicações em ambientes cloud ou tradicionais. Essas soluções ajudam as organizações a alcançar níveis mais elevados de serviços através de análises de transações, diagnóstico de problemas, análise de performance, etc.

`Continuous Delivery` – Prática que tem como objetivo, garantir que o novo código pode ser implantado no ambiente de produção a qualquer momento. É composta por todas as outras fases Continuous Integration, Continuous Inspection, Continuous Testing, Continuous Deployment e Continuous Monitoring.


### Arquitetura Proposta

A arquitetura do ecossistema DevOps proposto nesses artigos tem por objetivo:

* Gerenciar o ciclo de vida e saúde das aplicações Java.
* Obter descrição detalhada das aplicações; 
* Gerenciamento das dependências; 
* Automatização do processo de build; 
* Zero downtime deployment;
* Coleta de métricas que possam mensurar a qualidade do código fonte; 
* Teste de Carga automatizado;
* Teste de Aceitação;
* Monitoramento 24x7;
* Monitoramento de performance;


![Pipeline DevOps](/images/201510-devops-00.png)


## Provisionamento com Vagrant, Virtual Box e Ansible

Vagrant é um software que abstrai provedores de máquinas virtuais como VMWare, VirtualBox, KVM, entre outros. Além de possuir um DSL baseado em Ruby, o Vagrant oferece uma grande gama de posibilidades para integração com sistemas de gerenciamento de configuração como: Puppet, Chef, Ansible, etc.

VirtualBox é um programa de máquina virtual, tecnologia que simula um computador dentro de outro. Com ele é possível utilizar um sistema Linux dentro de outro sistema operacional.

Ansible é um sistema de automação de configuração desenvolvido em Python que permite descrever procedimientos utilizando o formato `YAML`.

### Preparando o Ambiente

A instalação do Vagrant é realizada utilizando o pacote `vagrant_1.8.1_x86_64.rpm` disponivel em [vagrant](https://www.vagrantup.com). 

Execute o pacote e verifique a versão instalada:

{% highlight bash %}
[mmagnani@linux DevOps]$ sudo rpm -Uvh vagrant_1.8.1_x86_64.rpm
[mmagnani@linux DevOps]$ vagrant -v
Vagrant 1.8.1
{% endhighlight %}

Instale também os pacotes para a integração com S.O:

{% highlight bash %}
[mmagnani@linux DevOps]$ sudo yum -y install libxslt-devel libxml2-devel libguestfs-tools-c
[mmagnani@linux DevOps]$ vagrant plugin install vagrant-nfs_guest
{% endhighlight %}

Observe os plugins instalados:

{% highlight bash %}
[mmagnani@linux DevOps]$ vagrant plugin list
vagrant-nfs_guest (0.1.8)
vagrant-share (1.1.5, system)
{% endhighlight %}

Para instalar o Ansible execute os seguintes comandos:

{% highlight bash %}
[mmagnani@linux DevOps]$ sudo yum -y install ansible
[mmagnani@linux DevOps]$ ansible --version
ansible 1.9.2
configured module search path = None
{% endhighlight %}


### Provisionando o Primeiro Servidor

Escolha um diretório no sistema operacional para a criação das configurações do projeto com por exemplo `/home/mmagnani/DevOps`.

Dentro desse diretório execute o comando vagrant init para provisionar o primeiro Boxe `(VM)` através do repositorio publico chamado Atlas Hashcorp.

{% highlight bash %}
[mmagnani@linux DevOps]$ vagrant init centos/7
{% endhighlight %}

Esse comando criou um arquivo chamado `VagrantFile` com uma estrutura inicial com comentários explicando cada opção. Inicialmente serão utilizadas apenas algumas dessas opções, sendo assim deixe o arquivo `VagrantFile` como abaixo:

{% highlight ruby %}
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    config.vm.box = "centos/7"

    config.vm.define :gitlab do |gitlab_config|
        gitlab_config.vm.hostname = "gitlab"
        gitlab_config.vm.network :private_network, :ip => "192.168.90.10"
    end
end
{% endhighlight %}

Para inicializar o Box utilize o comando `vagrant up`:

{% highlight bash %}
[mmagnani@linux DevOps]$ vagrant up 
{% endhighlight %}

Faça login no novo servidor e verifique a versão:

{% highlight bash %}
[mmagnani@linux DevOps]$ vagrant ssh gitlab
[vagrant@localhost ~]$ sudo su -
[root@localhost ~]# cat /etc/redhat-release
CentOS Linux release 7.1.1503 (Core)
{% endhighlight %}

O primeiro servidor já está disponível.


### Provisionando o GitLab CE

Quando novos projetos são iniciados, uma das primeiras dúvidas que surgem é: onde os arquivos desse projeto ficarão armazenados? Quando o projeto é desenvolvido por equipes muito grandes é necessário que ele esteja acessível a todos os desenvolvedores, com as devidas permissões e seus respectivos grupos, para evitar sobreposições nas altrerações ou incompatibilidade nas versões.
Para esse cenário um sistema de controle de versão pode ser utilizado, como por exemplo o Git, que é um sistema de controle de versão distribuído com ênfase em velocidade. Hoje em dia o Git é utilizado em muitos projetos de código aberto no mundo todo, sendo um dos maiores e mais importantes o kernel do Linux. Observe também que a grande maioria dos projetos da Red Hat atualmente utiliza o Git como controle de versão, pois os engenheiros e mantenedores dos produtos estão espalhados por todo o mundo.


Git e GitHub são ferramentas incríveis que fazem a gestão e administração dos repositórios Git e suas permissões. São excelentes, para a escrita de software `open source`, mas ao escrever software de código fechado faz-se necessário o uso de repositórios privados. Então como fazer para obter o controle, flexibilidade e facilidade de uso de algo como Github ou BitBucket, sem hospedar seus repositórios git em servidores fora do seu controle?

Pode-se utilizar o `GitLab CE`. O GitLab CE fornece uma interface simples, mas poderosa baseada em uma aplicação web para gerenciar os repositórios Git. Também pode controlar as permissões, criar grupos e auditar as alterações, controlando de maneira eficiente todas as iterações com os repositórios Git. A pagína oficial do projeto é: [gitlab](https://about.gitlab.com).

### Preparando a Máquina do Desenvolvedor


Para facilitar o desenvolvimento e configurações, adicione os IPs e nomes no arquivo `/etc/hosts`:

{% highlight bash %}
192.168.90.10 gitlab
192.168.90.20 jenkins
{% endhighlight %}

Ainda na maquina do desenvolvedor instale o `Git`:

{% highlight bash %}
[mmagnani@localhost DevOps]$ sudo yum install git -y
{% endhighlight %}

Verifique se foi instalado corretamente:

{% highlight bash %}
[mmagnani@localhost DevOps]$ git --version
git version 1.7.1
{% endhighlight %}

Após essa verificação, configure algumas informações do desenvolvedor para que sejam identificadas corretamente no repositório. Veja abaixo:

{% highlight bash %}
[mmagnani@localhost DevOps]$ git config --global user.name "Mauricio Magnani Jr"
[mmagnani@localhost DevOps]$ git config --global user.email "msmagnanijr@gmail.com"
{% endhighlight %}

Crie também uma chave SSH para ser utilizada posteriormente no GitLab: 

{% highlight bash %}
[mmagnani@linux DevOps]$ ssh-keygen -t rsa -b 4096 -C "msmagnanijr@gmail.com"
{% endhighlight %}


### Instalando GitLab CE

A instalação do GitLab CE será realizada utilizando Ansible. Sendo assim, crie um playbook chamado `gitlab.yml` no diretorio `/home/mmagnani/Workspace/DevOps` e deixe-o como abaixo:


{% highlight yaml %}
- hosts: gitlab
  sudo: True
  user: vagrant
  tasks:
    - name: "Atualiza pacotes e Instala OpenSSH Server"
      yum: name=openssh-server state=latest

    - name: "Instala o Curl"
      yum: name=curl state=latest

    - name: "Habilita o SSHD"
      shell: sudo systemctl enable sshd.service

    - name: "Inicia o SSHD"
      shell: sudo systemctl start sshd.service

    - name: "Instala Postfix"
      yum: name=postfix state=latest

    - name: "Habilita o Postfix"
      shell: sudo systemctl enable postfix

    - name: "Inicia o Postfix"
      shell: sudo systemctl start postfix

    - name: "Download do instalador GitLab"
      shell: curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

    - name: "Instala GitLab"
      shell: sudo yum -y install gitlab-ce

    - name: "Configura e Inicia o GitLab"
      shell: sudo gitlab-ctl reconfigure

    - name: "Adiciona o IP do Jenkins"
      lineinfile: 'dest=/etc/hosts line="192.168.90.20 jenkins"'
{% endhighlight %}

Atualize também as configurações do `VagrantFile` para utilizar o `playbook`:

{% highlight ruby %}

VAGRANTFILE_API_VERSION = "2"
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "centos/7"

  config.vm.define :gitlab do |gitlab|
    gitlab.vm.hostname = "gitlab"
    gitlab.vm.network :private_network, :ip => "192.168.90.10"

    gitlab.vm.provision "ansible" do |ansible|
          ansible.playbook = "gitlab.yml"
          ansible.verbose = "vvvv"
    end
 
    gitlab.vm.provider :virtualbox do |domain|
      domain.memory = 1024
      domain.cpus = 2
    end

  end

end
{% endhighlight %}

Para iniciar a configuração com o playbook criado, poderia ser utilizado o comando `vagrant provision` que iria atualizar as configurações. Como a criação de um Box se tornou uma tarefa trivial, basta apenas destruir com `vagrant destroy` e recriar com `vagrant up`:

{% highlight bash %}
[mmagnani@linux DevOps]$ vagrant destroy
[mmagnani@linux DevOps]$ vagrant up
{% endhighlight %}

Terminada a instalação com sucesso, navegue até a URL [gitlab](http://gitlab) e faça o login com o usuário e senha padrão:

* Username: root
* Password: 5iveL!fe

![](/images/201510-devops-01.png)

Essa senha deve ser alterada no primeiro acesso.

A instalação do GitLab está finalizada!

No próximo capítulo será abordada a criação de uma aplicação básica, bem com o envio para o repositório do GitLab.

## Observações

**Agradecimento** Gostaria de agradecer a Hevellyn Natasha ( Technical Support System Specialist na IBM ) pela excelente introdução ao DevOps desse post.

[DevOps - Uma Abordagem Prática - Capítulo I](http://mmagnani.me/2015/10/21/devops-provisionamento-1)

[DevOps - Uma Abordagem Prática - Capítulo II](http://mmagnani.me/2015/10/22/devops-provisionamento-2)

[DevOps - Uma Abordagem Prática - Capítulo III](http://mmagnani.me/2015/10/25/devops-provisionamento-3)

[DevOps - Uma Abordagem Prática - Capítulo IV](http://mmagnani.me/2015/10/26/devops-provisionamento-4)