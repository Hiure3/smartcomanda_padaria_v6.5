# SmartComanda – Ativar o login seguro (Firebase Authentication + regras)

Ordem importante: **primeiro** configure e teste o login, **por último** cole as regras.
Se colar as regras antes de o login estar funcionando, o sistema para de ler/gravar dados.

## 1) No console do Firebase (projeto smartcomanda-df58e)
1. **Authentication > Sign-in method**: ative **E-mail/senha** e **Anônimo**.
2. **Configurações do projeto (engrenagem) > Geral**: copie a **Chave de API da Web**.
3. **Authentication > Users > Add user**: crie o seu usuário gerente (e-mail + senha). Copie o **UID** dele.
4. **Realtime Database > Dados**: crie o caminho `staff/<UID copiado>` com estes campos:
   - `name` (texto): seu nome
   - `email` (texto): o mesmo e-mail
   - `role` (texto): `Gerente`
   - `active` (booleano): `true`
   (O console do Firebase ignora as regras, por isso este primeiro cadastro só pode ser feito aqui.)

## 2) Nos arquivos
Abra `index.html` **e** `comanda_cliente.html` e preencha, no começo do script, a linha:
`const SC_API_KEY = '';` → `const SC_API_KEY = 'SUA_CHAVE_DE_API_DA_WEB';`
(a chave é a mesma nos dois arquivos). Publique os dois arquivos juntos.

## 3) Teste ANTES de colar as regras
- No PC da loja: entre com o e-mail e a senha do gerente. A tela de login não deve mais mostrar o aviso "Modo antigo".
- Abra uma mesa, escaneie o QR no celular, mande uma mensagem no chat e faça uma encomenda.
- Em CRM & Acessos cadastre um colaborador (e-mail + senha provisória); ele entra e é obrigado a trocar a senha.

## 4) Colar as regras (só depois do passo 3 funcionar)
1. **Realtime Database > Regras**: copie e guarde o texto atual (para voltar atrás, se precisar).
2. Cole o conteúdo de `firebase-rules.json` e clique em **Publicar**.
3. Use o **Simulador de regras** do console para conferir: (a) leitura de `/tables` sem login deve ser **negada**;
   (b) leitura de `/encomendas` por um usuário anônimo deve ser **negada**.
4. Refaça o teste do passo 3. Se algo parar, restaure as regras antigas e me avise com a mensagem de erro.

## O que muda na prática
- Funcionários entram com e-mail e senha; **cargo e permissões vêm do banco** (`staff/<uid>`) e as regras impedem
  que alguém sem o cargo leia/grave (ex.: só Gerente lê Exclusões e Backups; o registro de exclusões só aceita novos itens).
- O app do cliente usa login anônimo e só acessa a **própria comanda** (chat, finalizar, avaliação e pedidos criados por ele).
  Ele não consegue listar mesas nem ler pedidos/mensagens de outros.
- A loja publica, para cada comanda aberta, uma cópia só dela (`mesa_publica`, `sessoes`, `acessos`). O QR precisa ter o `sid`.
- O identificador da comanda (`sid`) agora é aleatório e longo; funciona como a "chave" do QR.

## Limitações a conhecer
- Cliente que troca de aparelho (ou limpa os dados do navegador) **não** vê no app os pedidos antigos dele pelo número
  (o banco só entrega o pedido ao login que o criou). Na loja, tudo continua visível.
- Sem internet, o login novo não funciona (precisa do Firebase). Quem já está logado continua usando.
- Qualquer pessoa pode criar uma conta de e-mail no Firebase, mas sem o cadastro em `staff/<uid>` ela não tem nenhum acesso.
- As regras não foram testadas contra um Firebase real aqui: use o Simulador (passo 4) antes de confiar nelas.
