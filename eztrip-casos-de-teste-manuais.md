# EzTrip: Casos de Teste Manuais

## Objetivo

Este documento reúne a suíte manual de testes do EzTrip para avaliação acadêmica.

## Massa de teste recomendada

Para executar a suíte manual com clareza, recomenda-se criar pelo menos estes usuários:

- **Usuário A**: organizador e administrador da viagem
- **Usuário B**: participante convidado
- **Usuário C**: usuário opcional para testes extras de gastos e remoção

Sugestão de e-mails:

- `prof.a@eztrip.local`
- `prof.b@eztrip.local`
- `prof.c@eztrip.local`

Sugestão de senha forte válida:

- `Teste@123`

## Padrão recomendado de execução visual

Para padronizar a avaliação manual, recomenda-se que todos os testes sejam executados com o navegador em modo de inspeção responsiva.

Configuração sugerida:

- abrir o DevTools/Inspecionar do navegador durante os testes
- ativar o layout mobile/responsivo
- selecionar o dispositivo `iPhone SE`
- usar a resolução `375px x 667px`

Antes de iniciar qualquer caso desta suíte, recomenda-se manter o DevTools aberto em modo mobile com preset `iPhone SE (375px x 667px)`.

## Padrão recomendado para testes com mais de um usuário

Sempre que o caso de teste envolver colaboração, aprovação, convite, notificações ou atualização em tempo real, recomenda-se:

- usar duas sessões simultâneas
- manter o Usuário A em um navegador
- manter o Usuário B em outro navegador ou em uma janela/aba anônima

Isso permite validar corretamente o comportamento colaborativo e os eventos em real time sem encerrar a sessão do primeiro usuário.

## 1. Autenticação

### CT-01. Cadastro de novo usuário

- **Descrição**: validar que um novo usuário consegue se cadastrar com dados válidos.
- **Pré-condições**:
  - ambiente Docker em execução
  - MailPit acessível em `http://localhost:8025`
- **Passos**:
  1. Acesse `http://localhost/register`.
  2. Preencha nome, e-mail e senha forte.
  3. Confirme a senha.
  4. Clique em criar conta.
  5. Observe o redirecionamento para a tela de verificação de e-mail.
- **Resultado esperado**:
  - a conta é criada com sucesso
  - o sistema redireciona para a página de verificação de e-mail
  - um e-mail de verificação aparece no MailPit

### CT-02. Cadastro com senha fraca

- **Descrição**: validar as regras de senha do formulário.
- **Pré-condições**:
  - ambiente em execução
- **Passos**:
  1. Acesse `http://localhost/register`.
  2. Preencha nome e e-mail válidos.
  3. Informe uma senha simples, como `12345678`.
  4. Repita a mesma senha no campo de confirmação.
  5. Envie o formulário.
- **Resultado esperado**:
  - o cadastro não é concluído
  - a tela informa que a senha precisa conter letra maiúscula, minúscula, número e símbolo

### CT-03. Cadastro com confirmação de senha divergente

- **Descrição**: validar o bloqueio do cadastro quando a confirmação de senha não corresponde.
- **Pré-condições**:
  - ambiente em execução
- **Passos**:
  1. Acesse `http://localhost/register`.
  2. Preencha os dados válidos.
  3. Informe senhas diferentes nos campos de senha e confirmação.
  4. Envie o formulário.
- **Resultado esperado**:
  - o cadastro não é concluído
  - o formulário indica que a confirmação deve ser igual à senha

### CT-04. Confirmação de e-mail via MailPit

- **Descrição**: validar a ativação da conta por link enviado por e-mail.
- **Pré-condições**:
  - usuário recém-cadastrado
  - e-mail de verificação disponível no MailPit
- **Passos**:
  1. Abra `http://localhost:8025`.
  2. Localize o e-mail enviado para o usuário.
  3. Abra a mensagem.
  4. Clique no link de confirmação.
  5. Aguarde o redirecionamento.
- **Resultado esperado**:
  - o sistema confirma o e-mail com sucesso
  - o usuário é redirecionado para a tela de login

### CT-05. Reenvio de e-mail de verificação

- **Descrição**: validar o reenvio do e-mail de verificação quando necessário.
- **Pré-condições**:
  - usuário cadastrado e ainda não confirmado
- **Passos**:
  1. Acesse `http://localhost/login`.
  2. Tente entrar com o e-mail ainda não verificado.
  3. Observe o redirecionamento para a página de verificação.
  4. Clique em reenviar e-mail de verificação.
  5. Abra o MailPit.
- **Resultado esperado**:
  - o sistema informa que um novo e-mail foi enviado
  - uma nova mensagem aparece no MailPit

### CT-06. Login com conta confirmada

- **Descrição**: validar o acesso ao sistema após confirmação do e-mail.
- **Pré-condições**:
  - usuário com e-mail confirmado
- **Passos**:
  1. Acesse `http://localhost/login`.
  2. Informe e-mail e senha válidos.
  3. Clique em entrar.
- **Resultado esperado**:
  - o login é concluído
  - o usuário é direcionado para `/trips`

### CT-07. Login inválido

- **Descrição**: validar a rejeição de credenciais incorretas.
- **Pré-condições**:
  - usuário existente e confirmado
- **Passos**:
  1. Acesse `http://localhost/login`.
  2. Informe o e-mail correto e uma senha incorreta.
  3. Clique em entrar.
- **Resultado esperado**:
  - o login é recusado
  - o usuário permanece na tela de login
  - é exibida mensagem de erro de credenciais inválidas

### CT-08. Recuperação de senha

- **Descrição**: validar o envio do e-mail de redefinição de senha.
- **Pré-condições**:
  - usuário existente
- **Passos**:
  1. Acesse `http://localhost/forgot-password`.
  2. Informe o e-mail do usuário.
  3. Envie o formulário.
  4. Abra o MailPit.
- **Resultado esperado**:
  - o sistema informa que o e-mail de recuperação foi enviado
  - o MailPit recebe uma mensagem com link de redefinição

### CT-09. Redefinição de senha

- **Descrição**: validar a troca de senha por link enviado por e-mail.
- **Pré-condições**:
  - e-mail de redefinição disponível no MailPit
- **Passos**:
  1. Abra a mensagem de recuperação no MailPit.
  2. Clique no link recebido.
  3. Defina uma nova senha forte.
  4. Confirme a nova senha.
  5. Envie o formulário.
  6. Volte para `http://localhost/login`.
  7. Entre com a nova senha.
- **Resultado esperado**:
  - a senha é alterada com sucesso
  - o login com a nova senha funciona

### CT-10. Logout

- **Descrição**: validar a saída da sessão.
- **Pré-condições**:
  - usuário autenticado
- **Passos**:
  1. Acesse a lista de viagens.
  2. Clique no avatar do usuário no canto superior.
  3. Escolha a opção de sair.
- **Resultado esperado**:
  - a sessão é encerrada
  - o sistema retorna para a tela de login

## 2. Configuração do usuário

### CT-11. Alteração de idioma

- **Descrição**: validar a troca de idioma disponível na interface.
- **Pré-condições**:
  - usuário autenticado
- **Passos**:
  1. Na tela de viagens, clique no avatar do usuário.
  2. Abra a opção de idioma.
  3. Selecione `English`.
  4. Salve.
  5. Em seguida, repita e volte para `Português`.
- **Resultado esperado**:
  - os textos da interface mudam conforme o idioma escolhido
  - o idioma permanece consistente após a alteração

## 3. Viagens

### CT-12. Criar viagem

- **Descrição**: validar a criação de uma viagem completa.
- **Pré-condições**:
  - Usuário A autenticado
- **Passos**:
  1. Acesse `http://localhost/trips`.
  2. Clique em `Plan a trip`.
  3. Informe nome da viagem.
  4. Informe destino.
  5. Selecione a data inicial e a data final.
  6. Avance para a etapa de permissões.
  7. Mantenha as permissões desejadas.
  8. Finalize a criação.
- **Resultado esperado**:
  - a viagem é criada com sucesso
  - o sistema abre a página principal da viagem

### CT-13. Visualizar viagem criada

- **Descrição**: validar a visualização do resumo da viagem.
- **Pré-condições**:
  - ao menos uma viagem criada
- **Passos**:
  1. Acesse `http://localhost/trips`.
  2. Clique em uma viagem existente.
- **Resultado esperado**:
  - a página da viagem exibe nome, destino e período
  - os atalhos para participantes, gastos e lista de bagagem ficam visíveis

### CT-14. Buscar viagem por nome ou destino

- **Descrição**: validar o filtro de busca da tela de viagens.
- **Pré-condições**:
  - ao menos duas viagens cadastradas
- **Passos**:
  1. Acesse `http://localhost/trips`.
  2. Digite parte do nome de uma viagem no campo de busca.
  3. Limpe o campo.
  4. Digite parte do destino de outra viagem.
- **Resultado esperado**:
  - a listagem é filtrada conforme o texto informado
  - apenas as viagens compatíveis permanecem visíveis

## 4. Participantes, convites e permissões

### CT-15. Gerar link de convite

- **Descrição**: validar a geração de link de convite pelo administrador.
- **Pré-condições**:
  - Usuário A autenticado
  - viagem existente
- **Passos**:
  1. Entre na viagem.
  2. Abra a tela de participantes.
  3. Clique em `Share invitation link`.
- **Resultado esperado**:
  - o sistema gera um link de convite
  - o link é exibido no drawer de compartilhamento

### CT-16. Solicitar acesso por link de convite

- **Descrição**: validar o fluxo de entrada em uma viagem por link.
- **Pré-condições**:
  - link de convite gerado
  - Usuário B cadastrado e confirmado
  - usar outra sessão do navegador para o Usuário B, preferencialmente em navegador diferente ou janela anônima
- **Passos**:
  1. Abra o link de convite em janela anônima ou outro navegador.
  2. Faça login com o Usuário B, se necessário.
  3. Na página pública do convite, clique em `Request access`.
- **Resultado esperado**:
  - a solicitação de acesso é registrada
  - o Usuário B volta para a área autenticada com mensagem de sucesso

### CT-17. Aprovar solicitação de acesso

- **Descrição**: validar a aprovação de entrada do participante.
- **Pré-condições**:
  - Usuário A autenticado na viagem
  - Usuário B já solicitou acesso
  - Usuário A e Usuário B devem permanecer abertos ao mesmo tempo, em navegadores diferentes ou com uso de janela anônima
- **Passos**:
  1. No navegador do Usuário A, abra a viagem.
  2. Vá para a tela de participantes.
  3. Abra a área de solicitações de acesso.
  4. Aprove a solicitação do Usuário B.
- **Resultado esperado**:
  - a solicitação desaparece da lista de pendências
  - o Usuário B passa a fazer parte da viagem
  - a lista de participantes é atualizada

### CT-18. Rejeitar solicitação de acesso

- **Descrição**: validar a rejeição de uma solicitação de entrada.
- **Pré-condições**:
  - nova solicitação de acesso pendente
- **Passos**:
  1. No navegador do administrador, abra a área de solicitações.
  2. Rejeite a solicitação.
- **Resultado esperado**:
  - a solicitação é removida da lista
  - o solicitante não entra na viagem

### CT-19. Remover participante da viagem

- **Descrição**: validar a remoção de um participante por administrador.
- **Pré-condições**:
  - Usuário B já faz parte da viagem
  - Usuário A é administrador
- **Passos**:
  1. Abra a tela de participantes como Usuário A.
  2. No participante desejado, abra o menu lateral.
  3. Escolha remover.
- **Resultado esperado**:
  - o participante é removido da viagem
  - ele deixa de aparecer na lista de participantes

### CT-20. Comportamento de permissões para convite e aprovação

- **Descrição**: validar permissões dos participantes não administradores.
- **Pré-condições**:
  - criar uma nova viagem com as permissões `Invite other participants` e `Accept invitations` desativadas
  - Usuário B deve entrar nessa viagem
- **Passos**:
  1. Como Usuário A, crie uma nova viagem e desligue as duas permissões citadas.
  2. Gere o link de convite como administrador.
  3. Faça o Usuário B entrar na viagem.
  4. Acesse a tela de participantes com o Usuário B.
- **Resultado esperado**:
  - o Usuário B não visualiza o botão de compartilhar convite
  - o Usuário B não visualiza a área de solicitações de acesso
  - o administrador continua com acesso total

### CT-21. Restrição de API para gasto sem permissão

- **Descrição**: validar uma permissão aplicada no backend, usando a documentação da API.
- **Pré-condições**:
  - viagem criada com `Expenses.Add` desativada para membros
  - Usuário B participante dessa viagem
  - Usuário B autenticado
- **Passos**:
  1. Acesse `http://localhost:5000/swagger`.
  2. Faça login com o Usuário B pelo endpoint `POST /auth/login`.
  3. Copie o `accessToken` retornado.
  4. Clique em `Authorize` na interface da API e cole o token como `Bearer <token>`.
  5. Tente executar `POST /trips/{tripId}/expenses` para a viagem em que `Expenses.Add` está desativada.
- **Resultado esperado**:
  - a API recusa a criação do gasto por falta de permissão

## 5. Notificações

### CT-22. Receber notificação de solicitação de acesso

- **Descrição**: validar o recebimento de notificação para quem pode aprovar entradas.
- **Pré-condições**:
  - Usuário A autenticado na viagem
  - Usuário B envia uma nova solicitação de acesso
  - os dois usuários devem estar ativos em sessões diferentes ao mesmo tempo
- **Passos**:
  1. Mantenha o Usuário A com a viagem aberta.
  2. Em outro navegador, faça o Usuário B solicitar acesso.
  3. Observe o ícone de notificações do Usuário A.
- **Resultado esperado**:
  - o contador de notificações é incrementado
  - a nova notificação aparece no drawer de notificações

### CT-23. Marcar notificação individual como lida

- **Descrição**: validar a leitura individual da notificação.
- **Pré-condições**:
  - existe ao menos uma notificação não lida
- **Passos**:
  1. Abra o drawer de notificações.
  2. Clique em uma notificação não lida.
- **Resultado esperado**:
  - a notificação muda de status visualmente
  - o contador de não lidas é reduzido

### CT-24. Marcar todas as notificações como lidas

- **Descrição**: validar a ação em massa do drawer de notificações.
- **Pré-condições**:
  - existe mais de uma notificação não lida
- **Passos**:
  1. Abra o drawer de notificações.
  2. Abra o menu de opções.
  3. Clique em `Mark all as read`.
- **Resultado esperado**:
  - todas as notificações ficam marcadas como lidas
  - o contador de não lidas zera

## 6. Gastos

### CT-25. Criar gasto

- **Descrição**: validar o cadastro de gasto compartilhado.
- **Pré-condições**:
  - viagem com pelo menos dois participantes
- **Passos**:
  1. Entre na viagem.
  2. Abra `Expenses`.
  3. Clique em adicionar gasto.
  4. Preencha título, descrição, categoria, moeda e valor.
  5. Selecione os participantes envolvidos.
  6. Salve.
- **Resultado esperado**:
  - o gasto é criado com sucesso
  - ele aparece na listagem de gastos
  - o gráfico/resumo é atualizado

### CT-26. Visualizar detalhe de gasto

- **Descrição**: validar a abertura do detalhamento de um gasto.
- **Pré-condições**:
  - existe ao menos um gasto cadastrado
- **Passos**:
  1. Abra a tela de gastos.
  2. Clique em um gasto da lista.
- **Resultado esperado**:
  - o sistema mostra título, categoria, valor, pagador e participantes envolvidos

### CT-27. Editar gasto

- **Descrição**: validar alteração de um gasto existente.
- **Pré-condições**:
  - existe ao menos um gasto cadastrado
- **Passos**:
  1. Abra o detalhe do gasto.
  2. Clique em editar.
  3. Altere algum dado, como título ou valor.
  4. Salve.
- **Resultado esperado**:
  - o gasto é atualizado
  - a listagem reflete os novos dados

### CT-28. Excluir gasto

- **Descrição**: validar remoção definitiva de um gasto.
- **Pré-condições**:
  - existe ao menos um gasto cadastrado
- **Passos**:
  1. Abra o detalhe do gasto.
  2. Clique em excluir.
  3. Confirme a exclusão.
- **Resultado esperado**:
  - o gasto é removido da lista
  - os totais da tela são recalculados

### CT-29. Filtrar gastos do usuário

- **Descrição**: validar o filtro `My expenses`.
- **Pré-condições**:
  - existe pelo menos um gasto do usuário e um gasto de terceiros
- **Passos**:
  1. Abra a tela de gastos.
  2. Clique em `My expenses`.
  3. Depois volte para `All expenses`.
- **Resultado esperado**:
  - o filtro mostra apenas gastos em que o usuário participa
  - ao voltar para `All expenses`, a lista completa reaparece

## 7. Lista de bagagem

### Observação importante do módulo

- a lista de bagagem é individual por participante dentro da viagem
- não se trata de uma lista compartilhada entre todos os participantes

### CT-30. Aplicar template inicial da lista de bagagem

- **Descrição**: validar a criação automática da lista inicial do participante.
- **Pré-condições**:
  - viagem sem categorias de bagagem criadas para o participante atual
- **Passos**:
  1. Abra `Packing list`.
  2. Na tela vazia, clique para aplicar o template inicial.
- **Resultado esperado**:
  - categorias padrão são criadas para o participante atual
  - itens iniciais aparecem na lista
  - o progresso passa a ser exibido

### CT-31. Criar categoria manual na lista de bagagem

- **Descrição**: validar a criação de uma categoria personalizada.
- **Pré-condições**:
  - viagem existente
- **Passos**:
  1. Abra `Packing list`.
  2. Clique para adicionar categoria.
  3. Escolha a criação manual.
  4. Informe um nome como `Eletrônicos`.
  5. Confirme.
- **Resultado esperado**:
  - a nova categoria aparece na lista do participante atual
  - a categoria fica pronta para receber itens

### CT-32. Adicionar item na bagagem

- **Descrição**: validar inclusão de novo item em uma categoria.
- **Pré-condições**:
  - existe pelo menos uma categoria na lista do participante
- **Passos**:
  1. Dentro de uma categoria, localize o campo vazio de novo item.
  2. Digite um item, como `Carregador`.
  3. Saia do campo.
- **Resultado esperado**:
  - o item é criado
  - permanece um novo campo vazio para próximo item

### CT-33. Marcar item como pronto

- **Descrição**: validar a atualização do progresso da bagagem.
- **Pré-condições**:
  - existe pelo menos um item na lista
- **Passos**:
  1. Marque a caixa de seleção de um item.
- **Resultado esperado**:
  - o item fica marcado como concluído
  - o contador e o progresso são atualizados

### CT-34. Editar item da bagagem

- **Descrição**: validar a edição do texto de um item.
- **Pré-condições**:
  - existe pelo menos um item na lista
- **Passos**:
  1. Clique sobre o nome do item.
  2. Altere o texto.
  3. Saia do campo.
- **Resultado esperado**:
  - o novo texto é salvo e permanece visível

### CT-35. Remover item da bagagem

- **Descrição**: validar a exclusão de um item.
- **Pré-condições**:
  - existe pelo menos um item na lista
- **Passos**:
  1. Remova um item usando o controle disponível na linha do item.
- **Resultado esperado**:
  - o item é removido da categoria
  - o progresso é recalculado

### CT-36. Remover categoria da bagagem

- **Descrição**: validar a exclusão de categoria inteira.
- **Pré-condições**:
  - existe pelo menos uma categoria na lista
- **Passos**:
  1. Na categoria desejada, use a ação de remover categoria.
- **Resultado esperado**:
  - a categoria desaparece da lista
  - os totais gerais da bagagem são atualizados

## 8. Fluxos de tempo real

### CT-37. Atualização em tempo real ao aprovar participante

- **Descrição**: validar atualização sem recarregar a página.
- **Pré-condições**:
  - Usuário A e Usuário B logados em navegadores diferentes, ou em navegador normal e janela anônima
  - solicitação de acesso pendente
- **Passos**:
  1. Deixe o Usuário B com a aplicação aberta.
  2. Como Usuário A, aprove a solicitação de acesso.
  3. Observe a sessão do Usuário B sem atualizar a página.
- **Resultado esperado**:
  - o sistema do Usuário B recebe a atualização da entrada na viagem

### CT-38. Atualização em tempo real ao remover participante

- **Descrição**: validar a propagação da remoção na interface.
- **Pré-condições**:
  - Usuário A e Usuário B participando da mesma viagem
  - ambos devem permanecer conectados em sessões diferentes ao mesmo tempo
- **Passos**:
  1. Deixe ambos conectados.
  2. Como Usuário A, remova o Usuário B.
  3. Observe a interface do Usuário A e, se possível, do Usuário B.
- **Resultado esperado**:
  - a lista de participantes do administrador é atualizada
  - o evento é propagado em tempo real

## 9. Apoio via Swagger/Scalar para testes manuais

Em alguns cenários acadêmicos, pode ser útil validar a API diretamente.

### Quando usar

Recomenda-se usar `http://localhost:5000/swagger` quando for necessário:

- validar códigos HTTP
- confirmar uma restrição de permissão no backend
- reproduzir um fluxo sem depender da interface

### Como autenticar na documentação da API

1. Execute `POST /auth/login`.
2. Copie o `accessToken` retornado no corpo da resposta.
3. Clique em `Authorize`.
4. Informe `Bearer <accessToken>`.
5. Execute os endpoints protegidos desejados.
