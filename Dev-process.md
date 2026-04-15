# Começo

Decidi fazer um bot para o Discord que traduz ambientes multinacionais, pois vivemos um. A ideia de tornar isso minha entrada no Hackathon veio junto da implementação de gerar textos para brand em um cenário igualmente multinacional (como: comunicados, mensagens de publicidade), tornando a aplicação um catalisador de idiomas e ideias em ambos, um `Agentic Solution` e uma `Brand Tool`.


## Estágios iniciais

Comecei fazendo uma aplicação do Discord, segui o **[exemplo do próprio Discord](https://docs.discord.com/developers/quick-start/getting-started)** que tornou super simples de prosseguir com isto, não perdi muito tempo aqui.


## Ideia da LLM

Assim que estava com o bot operando pensei em meios que eu pudesse usar para traduzir, e como venho trabalhando em outros projetos que envolvem desenvolvimento com LLM optei por fazer uma que servisse para o processo de tradução.

A ideia era simples:

	Mensagem do bot –API→ LLM

## Usos iniciais da inteligência artificial (Claude Code) e melhorias da LLM

Após isso passei a usar o sistema que a Raptor paga para nós para treinar cenários com a LLM. Aí tive a ideia de adicionar o tal comando que gera textos já que tinha a LLM funcionando perfeitamente no âmbito da tradução.

Também configurei alguns comportamentos para facilitar meu processo de desenvolvimento como geração de relatórios de melhoria e habilidades de testes para o sistema (incluindo para o Docker, onde, por exemplo, instanciei uma imagem apenas para garantir que o modelo da minha LLM está operando como deveria).

## Interface web

Pedi ao agente que adicionasse uma interface web usando `React` pois gostaria que como `Brand Tool` agências que não possuíssem Discord pudessem também utilizar a aplicação, já que neste ponto não era mais um bot para Discord e sim uma solução de inteligência artificial empresarial. Peguei como exemplo as interfaces do chat GPT e do Claude Code com um sistema de histórico com um adicional de escolha para a personalidade da LLM (fiz um sistema de login para isto apenas para estar dentro dos requerimentos do Hackathon).

## Histórico

Para o sistema de histórico do bot queria que o comportamento fosse:

	Discord bot
			}→ Histórico em comum
	Interface web

Só que para isso eu teria que ter algum jeito de fazer com que o que fosse chamado no bot do Discord fosse identificado como do mesmo usuário X do backend da aplicação web, neste caso perguntei para a LLM qual seria o jeito mais correto de aplicar isto e ela fez com que o esquema funcionasse mais como:

	Discord bot
		}Passa o usuário do Discord no header→ Histórico em comum
	Interface web

E também criou uma tela de "perfil" e caso o usuário tivesse adicionado ao perfil dele o usuário do Discord ele seria então capaz de armazenar histórico tanto do que ele do bot do Discord quanto da aplicação web.

### SSE ou Web Socket

Eu também gostaria que o histórico fosse atualizado automaticamente e como nunca tinha feito um sistema de web socket por mim mesmo (apenas consumido) eu pedi para o agente IA me orientar sobre isso, e então ele me sugeriu que eu usasse SSE, pois exigiria menos etapas para o mesmo resultado.