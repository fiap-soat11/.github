# FIAP Pós Tech - Arquitetura de Software - Hackathon 
Repositórios do projeto em grupo "Tech Challenge" (Grupo 11 da SOAT11 de 2025) do curso de [Arquitetura de Software da FIAP Pós Tech](https://postech.fiap.com.br/curso/software-architecture/).

## Projeto  

A empresa FIAP X precisa avançar no desenvolvimento de um projeto de processamento de imagens. Em uma rodada de investimentos, a empresa apresentou um projeto simples que processa um vídeo e retorna as imagens dele em um arquivo .zip. 
Os investidores gostaram tanto do projeto, que querem investir em uma versão onde eles possam enviar um vídeo e fazer download deste zip. 

O objetivo do projeto é aplicar os tópicos relacionados à arquitetura de software para atender os requistos propostos:

 - A nova versão do sistema deve processar mais de um vídeo ao mesmo tempo 
 - Em caso de picos, o sistema não deve perder uma requisição 
 - O Sistema deve ser protegido por usuário e senha 
 - O fluxo deve ter uma listagem de status dos vídeos de um usuário 
 - Em caso de erro, um usuário pode ser notificado (e-mail ou outro meio de comunicação)

## Arquitetura

### HLD High-Level Design

![HLD](/hack/hld.png)


### LLD Low-Level Design

![LLD](/hack/lld.png)

### Arquitetura do Software

![DAS](/hack/DAS.png)


### Arquitetura de Dados

