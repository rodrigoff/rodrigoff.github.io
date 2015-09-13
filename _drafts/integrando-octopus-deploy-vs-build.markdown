---
title:  "Visual Studio Build + Octopus Deploy"
date:   2015-09-13 15:25:00
description: Automação de releases utilizando Octopus Deploy integrado ao Visual Studio Build
---

Seguindo as tendências de automatização, nesta série de posts vou mostrar como automatizar releases de aplicações web utilizando a plataforma [Octopus Deploy][OctopusDeploy] integrada à nova plataforma de build do TFS (Team Foundation Server), o [Team Foundation Build][TFBuild].

Octopus Deploy
--------------

O Octopus Deploy é uma plataforma de releases automatizados com foco em aplicações web feitas na plataforma .NET. A plataforma trabalha a partir de pacotes (.nupkg, formato utilizado pelo [NuGet][NuGet]), que são gerados pelo NuGet ou pelo [OctoPack][OctoPack], uma ferramenta própria da plataforma, que visa facilitar a criação e envio destes pacotes ao servidor de release.


### Gerando pacotes com o OctoPack

O primeiro passo para gerar pacotes pelo OctoPack, é instalar o mesmo nas aplicações que serão publicadas:

{% highlight powershell %}
Install-Package OctoPack
{% endhighlight %}

Feito isto, basta adicionar o seguinte parâmetro no processo de compilação (MSBuild):

{% highlight powershell %}
/p:RunOctoPack=true
{% endhighlight %}

A ferramenta então irá gerar um arquivo .nuspec (XML de definição das informações do pacote), caso o mesmo não exista, e, a partir deste, irá construir o pacote utilizando a versão encontrada nas propriedades da assembly gerada pela compilação, contendo as dependências necessárias para que o release seja feito com sucesso.

Repositório de pacotes
----------------------

Após ter gerado os pacotes das aplicações que serão publicadas, é necessário enviá-los para um repositório, que pode ser desde uma pasta compartilhada, até um servidor NuGet. Para facilitar ainda mais este passo, o Octopus já conta com este servidor embutido, bastando somente fazer o "push" do pacote para o mesmo:

{% highlight powershell %}
nuget push <Pacote gerado (.nupkg)> <ApiKey> <Endereço do servidor>
{% endhighlight %}

* A chave da API pode ser obtida a partir de um usuário do Octopus.

<IMAGEM DA PARTE DE API DO OCTOPUS>

Criação de um release
---------------------

Tendo os pacotes publicados, o próximo passo é gerar um novo release. Para isto, basta acessar o servidor do Octopus, encontrar o projeto desejado e utilizar a opção "Create release":

<IMAGEM DA CRIAÇÃO DE RELEASE PELO OCTOPUS>

Integrando com o Team Foundation Build
--------------------------------------



[OctopusDeploy]:https://octopusdeploy.com
[TFBuild]:https://msdn.microsoft.com/en-us/Library/vs/alm/Build/overview
[NuGet]:http://nuget.org
[OctoPack]:https://github.com/OctopusDeploy/OctoPack