# Plano de Testes – Lanchonete FIAP

## Objetivo

Este documento tem como objetivo definir as diretrizes, critérios e casos de teste para validar as funcionalidades da WebAPI do Totem de Lanchonete Fast Food, garantindo que todos os requisitos funcionais estejam implementados corretamente e que o sistema atenda às expectativas de qualidade.

## Funcionalidades a serem testadas

1. **Cadastro do Cliente**
    - Testar cadastro com dados válidos.
    - Testar cadastro com dados inválidos (campos obrigatórios ausentes, CPF inválido).
    - Testar cadastro de cliente já existente.

2. **Identificação do Cliente via CPF**
    - Testar identificação com CPF válido e cadastrado.
    - Testar identificação com CPF não cadastrado.
    - Testar identificação com CPF em formato inválido.
    - Testar identificação com Email válido e cadastrado.
    - Testar identificação com Email não cadastrado.
    - Testar identificação com Email em formato inválido.

3. **Criar, Editar e Remover Produtos**
    - Testar inclusão de produto no pedido com dados válidos.
    - Testar inclusão de produto no pedido com dados inválidos.
    - Testar edição de produto do pedido existente.
    - Testar edição de produto do pedido inexistente.
    - Testar remoção de produto do pedido existente.
    - Testar remoção de produto do pedido inexistente.

4. **Buscar Produtos por Categoria**
    - Testar lista de categorias.
    - Testar busca com categoria existente.
    - Testar busca com categoria inexistente.
    - Testar busca sem informar categoria.

5. **Fake Checkout (Finalização do Pedido)**
    - Testar envio de pedidos válidos para a fila de pedidos.
    - Testar envio de pedidos inexistentes.
    - Testar envio de pedido vazio.
    - Testar pagamento com sucesso
    - Testar pagamento recusado
    - Testar pagamento recusado 5 vezes
    - Testar cancelamento do pedido

6. **Listar os Pedidos**
    - Testar listagem de pedidos existentes.
    - Testar listagem quando não houver pedidos.

## Critérios de Aceitação

- Todas as funcionalidades devem ser testadas com dados válidos e inválidos.
- As respostas da API devem estar de acordo com o esperado (status codes, mensagens de erro, etc).
- Não deve haver falhas críticas ou bloqueadoras.

## Considerações Finais

Os testes serão realizados utilizando swagger, conforme necessário, e os resultados serão documentados para acompanhamento e validação das correções.
