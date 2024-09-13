## `Repositório Público de Projeto para o Hackaton ICP HUB BR`
#### Atenção: Esse repositório foi criado para a participação no Hackaton do HUB ICP BR, é de uso público e possui cunho educacional, portanto pode não conter todas as informações de futuras implementações do projeto.

# ICP Runes Control Panel

## Resumo

Esse projeto propõe-se a construir uma aplicação real de uso imediato para recompensar os membros da comunidade ICP HUB Brasil, auxiliar na propagação do conhecimento e visibilidade do HUB e incentivar outros projetos como comunidades e DAOs com esse mesmo objetivo.

## O Problema

A maioria das ferramentas hoje existentes para recompensar comunidades por engajamento e participação em eventos são off-chain, baseadas em sistemas de pontuação providos pelos próprios projetos de terceiros e limitadas às interações e redes sociais aceitas por eles.

## Nossa Visão

Nossa principal visão é unir a segurança e confiabilidade da rede do Bitcoin com a escalabilidade e programabilidade da rede ICP. Com isso, permitimos ao HUB ICP BR uma forma única, exclusiva e valiosa de recompensar os membros de sua comunidade. Nossa implementação permite o envio em massa dessas recompensas onchain para os membros da comunidade, além de permitir a programabilidade de outras ferramentas como bots de Discord e contratos automatizados para distribuição desses ativos. Nosso projeto se utiliza de um padrão de "tokens" fungíveis extremamente inovador e revolucionário na rede do Bitcoin, as Runas, que foram lançadas em 20/04/2024 (Halving) graças a implementação do protocolo Ordinals, que traz a possibilidade de registrar uma quantidade muito maior de dados nos Satoshis (menor fração de 1 Bitcoin), como textos, fotos, vídeos e tokens.

## A Solução

Nossa solução permite a utilização de ativos onchain para recompensar a comunidade da ICP HUB BR em eventos, torneios e sorteios promovidos pelo HUB, de maneira flexível, permitindo uma integração tecnológica e de cunho educacional, tanto em relação a desenvolvimento na rede do Bitcoin quanto na rede ICP.  
A partir da inscrição dessas runas, utilizamos a infraestrutura da Omnity Network para fazer uma ponte com o objetivo de representar esse ativos na rede da ICP e armazená-los em um canister de controle, que administra esses ativos e os distribui simultaneamente para vários usuários, de forma muito mais barata e escalável graças à estrutura da rede ICP.

## Funcionalidades Implementadas no Canister

#### Gerenciamento de Acesso 
- "Ownership" - permite que apenas a Account autorizada faça a distribuição de runas hospedadas no Canister.
A estrutura de gerenciamento de acesso permite que qualquer pessoa consulte o endereço do Principal autorizado, e também permite que ele faça a transferência da sua autorização para outrem.

#### Distribuição de Runas 
- Permite a distribuição facilitada e automatizada das Runas armazenadas no Canister.
A estrutura do FrontEnd proporciona um método direto e intuitivo para realizar a distribuição simultânea de Runas para uma quantidade virtualmente ilimitada de pessoas.

#### Integração com a Internet Identity
- Permite um Login rápido na plataforma, baseado na confiabilidade e segurança da Internet Identity.

## Como Fazemos 

#### - Passo 1:
Inscrição dos ativos do protocolo Runes na rede L1 do Bitcoin, representando a ICP HUB BR na maior e mais segura blockchain existente.

(usamos o app https://ordinalsbot.com/runes pela simplicidade e rapidez, como alternativa à inscrição direta em um nó de BTC)

#### - Passo 2:
Registro das Runas na rede da ICP utilizando a ferramena de bridge da infraestrutura Omnity Network - https://bridge.omnity.network/runes 

(verificável via cannister criado pela própria Omnity Network em um processo de travamento de ativos na camada 1 do Bitcoin e emissão na rede ICP com o padrão IRCR-1)

#### - Passo 3:
Registro do nosso próprio cannister para manipulação e distribuição das Runas/Tokens na rede ICP.

A principal ferramenta para criar, implementar e gerenciar os dapps para a plataforma da IC é a dfx (DFINITY command-line execution environment), que está contida na IC SDK (software development kit). 

Essa ferramenta não tem suporte nativo no Windows, portanto todo o desenvolvimento desse projeto foi centrado em uma configuração a partir de um ambiente WSL (Windows Subsystem for Linux).

Esse Canister possibilita uma distribuição automatizada das Runas armazenadas, bem como um controle de acesso para quem pode fazer essa distribuição (ownership).

#### Para o exemplo, vamos considerar que 5 pessoas ganharam uma competição de arte para a ICP Hub BR e estão elegíveis para receber uma premiação em Runas:

```bash
A – Aline – 16 Runas – xfznx-sgxv2-ty5tf-niw3v-6u34s-suz5w-xza2d-kwpwo-qq7fc-ug4oc-oqe

B – Bob – 8 Runas – 6fp7j-oxpeg-gwgjl-zvjsj-kmne4-wq6cw-wnh5h-tbfii-q2nvs-gwpvm-2ae

C – Carlos – 4 Runas – z3fut-ezcmm-f3qfz-4hxqz-7fmp7-bu74a-js3td-g5wr5-2ncqj-u5mwq-vqe

D – Denise – 2 Runas – pykrr-ecxve-bksoh-oipzw-hdpoj-djhdp-brkgd-h7h54-oincy-lu4wn-zqe

E – Edward – 1 Runa – mgc4g-k3pmg-wtkha-omup5-y7eom-ip4b5-lvjdx-qu5wr-nl6wm-vbhls-kae
```

#### Testando Localmente:

Antes de começar, é necessário que você tenha configurado um ambiente de desenvolvedor. Alguns tutoriais em relação à isso estão disponíveis da documentação da ICP.

#### Inicializar replica local:
```bash
dfx start --clean --background
```

#### Primeiramente vamos fazer o deploy das Runas como um ICRC-1 localmente:

#### Para isso, primeiro definimos um “Minter” e uma conta padrão “Default” – para onde as Runas serão mintadas:

```bash
dfx identity new minter
dfx identity use minter
export MINTER=$(dfx --identity anonymous identity get-principal)
dfx identity new default
dfx identity use default
export DEFAULT=$(dfx identity get-principal)
```

#### E então podemos dar o deploy no Canister. Nessa configuração, serão mintadas 100 Runas para o
#### Principal “Default” e a taxa de transferência será inicializada em 0,0001 tokens.

```bash
dfx deploy icrc1_ledger_canister --argument "(variant { Init =
record {
 token_symbol = \"ICRC1\";
 token_name = \"L-ICRC1\";
 minting_account = record { owner = principal \"${MINTER}\" };
 transfer_fee = 10_000;
 metadata = vec {};
 initial_balances = vec { record { record { owner = principal \"${DEFAULT}\"; };
10_000_000_000; }; };
 archive_options = record {
 num_blocks_to_archive = 1000;
 trigger_threshold = 2000;
 controller_id = principal \"${MINTER}\";
 };
 }
})"
```

#### Com as Runas mintadas, vamos fazer o deploy do nosso Canister, Frontend e também o Canister da Internet Identity, que permitirá a conexão com o usuário:

```bash
dfx deploy hackaton_project_backend
```

```bash
dfx deploy internet_identity
```

```bash
dfx deploy hackaton_project_frontend
```

#### Então podemos transferir Runas para o Canister. 
#### Nesse exemplo, vamos enviar 10 Runas:

```bash
dfx canister call icrc1_ledger_canister icrc1_transfer "(record {
 to = record {
 owner = principal \"$(dfx canister id hackaton_project_backend)\";
 };
 amount = 1_000_000_000;
})"
```

#### E nosso Setup Local está pronto! A partir daqui, tudo pode ser realizado diretamente no Frontend. Mas caso queira fazer os testes pelo console, aqui estão alguns exemplos:

Chamando a função “transferPrivilege” passando de argumento o Principal que queremos utilizar como “Owner”, que fará a distribuição das Runas. Nesse caso será o próprio Principal que estamos usando (“Default”):

```bash
dfx canister call hackaton_project_backend transferPrivilege "(principal \"$(dfx identity get-principal)\")"
```

Fazendo a distribuição das Runas da Denise e do Edward:
*Como proteção para eventuais erros, o Canister requer que a lista de Accounts e a lista de Amounts possua o mesmo tamanho.

```bash
dfx canister call hackaton_project_backend bulkTransfer "(record {
 amounts = vec {
 200_000_000;
 100_000_000;
 };
 toAccounts = vec {
 record { owner = principal \"pykrr-ecxve-bksoh-oipzw-hdpoj-djhdp-brkgd-h7h54-oincy-lu4wnzqe\" };
 record { owner = principal \"mgc4g-k3pmg-wtkha-omup5-y7eom-ip4b5-lvjdx-qu5wr-nl6wmvbhls-kae\" };
 };
})"
```
## Mídias do Projeto

- **Testando Canister Local**

![image](https://github.com/user-attachments/assets/65cfff03-8eea-46bf-92f4-a553d8be793a)

- **Login pela Internet ID**

![image](https://github.com/user-attachments/assets/737bb2ed-63c5-4e6d-9067-8813b9a0e023)


- **Transferência para Múltiplos Usuários Simultaneamente**

![image](https://github.com/user-attachments/assets/81449380-1453-4d1c-bec1-2b325292ac19)

- **Transferência de Privilégios (Ownership)**

![image](https://github.com/user-attachments/assets/1a0ed7fd-bbdc-43bd-8507-541169023878)

## Demonstração do Website

### Link: [Video Website Demo - PT-BR](https://drive.google.com/file/d/17XZamV7xKg2Hs9WDM7sDYC1fipLar9n9/view?usp=sharing)

## Time

#### Hiago Ferucci [X/Twitter](https://x.com/CobraX64), [LinkedIn](https://www.linkedin.com/in/marcelo-dussel-bb85a3203/)

#### Marcelo Dussel [LinkedIn](https://www.linkedin.com/in/marcelo-dussel-bb85a3203/)

#### Realização: Aegis Protocol [X/Twitter](https://x.com/DAOAegis), [Website](https://aegisprotocol.xyz/)
