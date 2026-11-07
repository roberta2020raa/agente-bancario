# Grain — CoffeeBank

Assistente definido em [RF01](requisitos/RF01.md) e
[DESIGN_RF01](requisitos/DESIGN_RF01.md).

## Rodar a demonstração no Windows

O ambiente virtual e as dependências já estão preparados neste computador.
Não é necessário ativar o ambiente virtual nem configurar uma chave OpenAI para a demo.

No **PowerShell**:

```powershell
Set-Location "C:\Users\Teste\agente bancario"
.\.venv\Scripts\python.exe -X utf8 -m grain.demo
```

No **Prompt de Comando (cmd.exe)**:

```bat
cd /d "C:\Users\Teste\agente bancario"
.venv\Scripts\python.exe -X utf8 -m grain.demo
```

Digite um comando por vez:

```text
saldo
extrato
últimos 30 dias
depósitos 2026-11-01 2026-11-07
saques 2026-11-01 2026-11-07
extrato 2026-11-01 2026-11-07
próxima página
transação demo-tx-001
atendente
sair
```

A interface reconhece comandos limitados; ainda não é uma conversa livre com um LLM.
Sem período, o Grain pergunta e sugere os últimos 30 dias, sem consultar automaticamente.
Os exemplos usam uma referência fictícia fixa: **07/11/2026**. Não representam dados atuais.

### Memória e retomada

- `pausar`: permite retomar a sessão sintética. Anote o UUID exibido ao iniciar.
- `sair`: encerra e revoga a sessão. EOF e Ctrl+C também encerram, sem retomada.
- Para retomar uma sessão **pausada**, substitua UUID_EXIBIDO pelo identificador exibido:

```bat
.venv\Scripts\python.exe -X utf8 -m grain.demo --resume "UUID_EXIBIDO"
```

A memória fica em `.sessions/demo`, sem criptografia. O UUID não autentica um usuário.
**Após retomar, informe novamente o tipo de consulta e o período.** Filtros, página,
tipo pendente e clarificação salvos não autorizam novas consultas, mesmo em um JSON válido.
Por exemplo: `saques 2026-11-01 2026-11-07`; só depois use `próxima página`.
A memória de filtros e paginação continua funcionando na sessão ativa. O histórico
sintético local é mantido como dado não autenticado, nunca como fonte de valores.

O armazenamento rejeita links/junctions nos componentes acessados e protege a troca
por renomeação durante I/O no Windows. Escritas usam temporário exclusivo e substituição
atômica; falhas interrompem a sessão. Não é isolamento de um usuário/SO comprometido.
Não utilize a mesma sessão em duas instâncias. Após falha abrupta ou erro de disco,
inicie uma nova sessão; a demo não remove bloqueios de segurança automaticamente.

## Estado e limites

**A demonstração é exclusivamente fictícia. Não digite dados pessoais ou credenciais.**

Ela utiliza Strands, quatro tools próprias e serviços locais em memória, somente leitura.
O encaminhamento humano é simulado e não aciona nenhuma pessoa ou canal real.
O modelo é determinístico e local: **não é OpenAI, não é um LLM e não gera inferência paga**.
Não há fallback automático de provedor. O módulo de demonstração não lê a chave do .env.

As consultas bancárias reais, autenticação, modelo OpenAI, política de dados,
retenção/expiração de produção, revisão independente e publicação continuam pendentes.
Não há servidor web nem repositório remoto criado.

Veja os [contratos e limites da demo](requisitos/DEMO_RF01.md) e o
[acompanhamento do plano](requisitos/PLANO_IMPLEMENTACAO_RF01.md), seção 9.

## Preparação de um novo ambiente Windows

O comando `python` do PATH deste computador apontava para o alias da Microsoft Store.
O launcher `py` e o executável da .venv foram validados:

```bat
py -m venv .venv
.venv\Scripts\python.exe -m pip install -e ".[dev]"
```

A instalação editável atual já foi validada. Não é necessário reinstalar para usar a demo.

## Diagnóstico administrativo de configuração

Este comando **não inicia o chat** e **não chama a OpenAI**:

```bat
.venv\Scripts\python.exe -m grain --project-root .
```

Para esse diagnóstico, preencha o .env a partir de .env.example fora da conversa:

- `OPENAI_API_KEY`: chave técnica; nunca envie pelo chat.
- `OPENAI_MODEL`: identificador a escolher, sem fallback automático.
- `SESSION_STORAGE_DIR`: caminho local dentro da raiz do projeto, padrão ./.sessions.

Campos vazios impedem a validação. Variáveis do processo prevalecem sobre o arquivo,
inclusive quando vazias. A leitura usa caminho explícito, não busca arquivos em diretórios
ancestrais, não expande variáveis e não modifica o ambiente do processo.
Não versionar segredos, ambiente virtual ou memória de sessão.

Validar configuração não comprova disponibilidade da chave, suporte a tools
ou autorização bancária. Dados reais exigem políticas aprovadas e integração autenticada.

## Verificação

```bat
.venv\Scripts\python.exe -m pytest -q -W error
.venv\Scripts\python.exe -m ruff check .
.venv\Scripts\python.exe -m ruff format --check .
.venv\Scripts\python.exe -m mypy
.venv\Scripts\python.exe -m pip check
```

A suíte usa dados sintéticos e bloqueia DNS e conexões diretas; permite apenas o
socketpair interno necessário ao asyncio no Windows. Verifica consultas, filtros,
paginação, recusas, uma repetição máxima, retorno inconsistente, bloqueio entre contas,
sanitização por gramática fechada, retomada, revogação, falha de disco e caminhos longos.

Revalidação após as correções: **200 testes aprovados e 3 exclusões justificadas**, de
203 casos, em duas execuções completas. Inclui os 76 casos adversariais originais
(73 aprovados, 3 exclusões), 95 testes de regressão e 32 testes adicionais de segurança.
Duas expectativas de retomada foram atualizadas com aprovação de Roberta; as asserções
dos ataques F-01/F-02 não foram alteradas. Ruff/formatação, Mypy strict e pip check passaram.
O fluxo de terminal e a retomada foram exercitados com dados fictícios por testes da CLI.

Detalhes e limites: [revalidação de segurança](requisitos/REVALIDACAO_SEGURANCA_RF01.md).
As exclusões cobrem SCA/proveniência, A2A ausente e LLM autônomo inexistente; não são aprovações.

Ambiente verificado: Windows, Python 3.14.6 e Strands 1.60.0. Outros sistemas/versões,
OpenAI real e segurança de produção ainda não foram homologados.
O adaptador de caminhos de LocalFileStorage é testado com essa versão fixada;
uma atualização do SDK exige repetir as regressões de persistência.
