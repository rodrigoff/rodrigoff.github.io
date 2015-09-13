---
title:  "Criando novas tasks para o TF Build vNext"
date:   2015-09-13 18:20:00
description: Criação e upload de novas tasks para o Team Foundation Build vNext
---

Recentemente comecei a utilizar a nova plataforma de build integrada ao TFS 2015 (ou Visual Studio Online), o Team Foundation Build vNext. Durante este período, tive a necessidade de criar novas tasks ou até mesmo alterar tasks já existentes e, portanto, vou falar sobre isso neste post.

Tasks do TF Build são simplesmente pequenos pacotes contendo scripts feitos em PowerShell ou NodeJS com um "acesso privilegiado" à plataforma de build e a determinadas funções.

Criando uma nova task
---------------------

A criação de uma task é feita pelo [tfx-cli (TFS Cross Platform Command Line Utility)][tfs-cli], uma ferramenta criada com o propósito de gerenciar extensões do TFS, de código aberto e feita pela própria Microsoft.

O primeiro passo para criar uma task, é instalar o tfx, para isso, é necessário ter o NodeJS instalado e configurado, e simplesmente instalar o pacote pelo npm:

{% highlight powershell %}
npm install -g tfx-cli
{% endhighlight %}

Feito isto, basta executar o seguinte comando:

{% highlight powershell %}
tfx build tasks create
{% endhighlight %}

E preencher os dados necessários:

{% highlight powershell %}
Enter short task name >
Enter friendly name >
Enter description >
Enter author >
{% endhighlight %}

Pronto, já temos nossa estrutura montada, que consiste em um arquivo de configurações (task.json), um ícone de tamanho 32x32 e 2 arquivos de execução (sample.js e sample.ps1):

{% highlight powershell %}
SampleTask
--icon.png
--sample.js
--sample.ps1
--task.json
{% endhighlight %}

Customizando uma task
---------------------

Agora que temos nossa estrutura criada, vamos para a parte de customização. Todo o processo de configuração da task é feita pelo arquivo task.json, criado na seção anterior pelo tfx.

Vou dividir a explicação destas configurações em 3 seções, configurações, parâmetros e execução:

### Configurações

{% highlight json %}
"id": "49095f70-5a51-11e5-87fd-6b6ad4e56e1e",
"name": "SampleTask",
"friendlyName": "Sample task",
"description": "Task de exemplo",
"author": "Rodrigo F. Fernandes",
"helpMarkDown": "Replace with markdown to show in help",
"category": "Utility",
"visibility": [
  "Build",
  "Release"
],
"demands": [],
"version": {
  "Major": "0",
  "Minor": "1",
  "Patch": "0"
},
"minimumAgentVersion": "1.83.0",
"instanceNameFormat": "Exibir mensagem: $(message)"
{% endhighlight %}

A maior parte das configurações é auto explicativa, porém, existem algumas menos óbvias, como a configuração "visibility", "demands" e "version" e "instanceNameFormat. Pela falta de uma documentação oficial até o momento, a seção a seguir foi feita baseada em observações durante a criação de algumas tasks customizadas.

* **Visibility**: possui 3 possíveis valores, "Build", "Release" e "Preview". O recomendado é utilizar o que foi definido como padrão pelo tfx, "Build" e "Release", para evitar futuros problemas. De acordo com algumas fontes, a opção "Preview" seria somente para uso interno, nos releases de preview do VSO.

* **Demands**: array de string contendo as demandas necessárias do agente. Caso sua task dependa de alguma ferramenta, variável de ambiente ou algo parecido, isto deve ser definido nesta configuração.

* **Version**: simplesmente a versão da task. A princípio parece ser uma configuração sem muita importância, porém, ao alterar algumas tasks customizadas, notei que, caso a versão não seja alterada, os agentes que já possuem a mesma instalada, não terão o código novo, então, é importante sempre atualizar a versão.

* **InstanceNameFormat**: formato do nome da task ao ser inserida em alguma configuração de build. No exemplo, após o usuário preencher o parâmetro "message" com o valor "Teste", o passo assumiria o nome "Exibir mensagem: Teste". Lembrando que o nome do build step pode ser alterado pelo usuário a qualquer momento dentro do TFS ou VSO.

### Parâmetros

Os parâmetros são definidos no objeto "inputs", um array contendo os campos que serão renderizados na configuração da task:

{% highlight json %}
"inputs": [
  {
    "name": "cwd",
    "type": "filePath",
    "label": "Working Directory",
    "defaultValue": "",
    "required": false,
    "helpMarkDown": "Current working directory when SampleTask is run."
  },
  {
    "name": "msg",
    "type": "string",
    "label": "Message",
    "defaultValue": "Hello World",
    "required": true,
    "helpMarkDown": "Message to echo out"
  }
],
{% endhighlight %}

Além destas, temos uma propriedade extremamente útil, a "visibleRule", que aceita uma expressão, que define se o campo será mostrado. Caso seja necessário exibir um campo somente quando uma opção for verdadeira esta opção deverá ser utilizada.

#### Tipos de parâmetros

Os parâmetros possuem alguns tipos que definem como serão renderizados em sua configuração:

* **string**: campo de texto básico;
* **multiLine**: campo de texto com múltiplas linhas;
* **filePath**: campo de texto com acesso ao repositório, com o objetivo de demonstrar um caminho;
* **boolean**: checkbox, para valores "true" ou "false";
* **pickList**: dropdown list com os valores definidos na propriedade auxiliar "options".

*Mais informações sobre os tipos de parâmetros podem ser encontradas em [vso-agent-tasks][vso-agent-tasks], explorando as tasks já existentes.*

#### Agrupamentos

Os campos também podem ser agrupados através da propriedade "group", que vincula o mesmo a um determinado grupo criado pela propriedade "groups":

{% highlight json %}
"groups": [
  {
   "name": "advanced",
   "displayName": "Advanced",
   "isExpanded": false
  }
],
"inputs": [
  {
    "name": "cwd",
    "type": "filePath",
    "label": "Working Directory",
    "defaultValue": "",
    "required": false,
    "helpMarkDown": "Current working directory when SampleTask is run.",
    "group": "advanced"
  }
],
{% endhighlight %}

### Execução

{% highlight json %}
"execution": {
  "Node": {
    "target": "sample.js",
    "argumentFormat": ""
  },
  "PowerShell": {
    "target": "$(currentDirectory)\\sample.ps1",
    "argumentFormat": "",
    "workingDirectory": "$(currentDirectory)"
  }
}
{% endhighlight %}

Esta é uma seção simples contém as instruções para execução da task. Caso a mesma tenha uma versão em NodeJS e PowerShell, preencha os 2 objetos, caso seja feita somente com uma tecnologia, basta remover o que não foi utilizado.


Publicando uma task
-------------------

Após a configuração da task, o próximo passo é a publicação, que pode ser feita pela ferramenta tfx.

O primeiro passo é autenticar-se no TFS, seja pelo VSO ou por uma instalação local do mesmo. Atualmente, a autenticação no VSO utiliza um token de acesso, que pode ser gerado através de seu perfil, pela opção "Personal access tokens", na parte de segurança. Caso seja utilizada uma instalação local do TFS, é necessário habilitar "Basic authentication", tanto no IIS, quanto no WebSite do TFS ([Mais informações][ConfiguringBasicAuth]).

{% highlight powershell %}
tfx login [<authType>]
{% endhighlight %}

Caso utilize uma instalação local do TFS, o parâmetro "authType" deve ser definido como "basic":

{% highlight powershell %}
tfx login --authType basic
{% endhighlight %}

Feito isto, será necessário informar a URL da coleção, além das informações de login.

O próximo passo é realizar o upload de fato, através do seguinte comando:

{% highlight powershell %}
tfx build tasks upload <taskPath>
{% endhighlight %}

Onde "taskPath" é o caminho no qual a task foi criada.
Pronto, agora temos nossa nova task pronta para ser utilizada.

![Add build step](/assets/images/posts/criando_tasks_tf_build_vnext/add_build_step.png)
![Configure build step](/assets/images/posts/criando_tasks_tf_build_vnext/configure_build_step.png)

[tfs-cli]:https://github.com/Microsoft/tfs-cli
[vso-agent-tasks]:https://github.com/Microsoft/vso-agent-tasks
[ConfiguringBasicAuth]:https://github.com/Microsoft/tfs-cli/blob/master/docs/configureBasicAuth.md