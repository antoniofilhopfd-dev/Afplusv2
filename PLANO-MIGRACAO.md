# Plano: AF+ da Netlify + planilhas para a Hostinger + MySQL

Rascunho para aprovação. Nada disso foi gravado em planilha, banco ou repositório.

## Decisões já tomadas (suas)
- Fonte oficial: só o MySQL. As planilhas ficam congeladas como arquivo.
- Um módulo por vez, começando por Alimentação; depois Corpo e Treinos.
- AF+ em `pessoal.antoniopfd.com`. O EVO MKT fica fora deste plano.
- Prototipar e mostrar prints antes de gravar; backup antes de mexer em planilha; merge só com a sua aprovação.

## Achados no código (`af-plus`, branch `main`)
- Dados: o app lê e grava via `src/lib/sheets-client.ts`, que chama o Apps Script (`SHEETS_API_URL`, `SHEETS_API_KEY`). Há cache por aba e um estado "recém-gravado" em memória.
- **Não encontrei login nenhum.** `current-user.ts` pega o primeiro usuário do SQLite e o `proxy.ts` só filtra rotas. Qualquer pessoa com a URL entra.
- Resquícios do SQLite/Prisma (`prisma/`, `db.ts`) ainda existem fora do menu.
- A preparação para a Hostinger (guia, `docs/hospedagem-hostinger.md`) não está no repositório; precisa ser refeita.
- Os scripts `Consumos.gs`, `Api.gs`, `Auditoria.gs` não estão no repo (só `apps-script/migrar-v11-para-v2.gs`); vêm dos seus prints.

## Etapas
1. **Hospedagem:** branch nova a partir da `main`, com o app pronto para Node.js Web Apps da Hostinger (porta, `npm start`, variáveis de ambiente) e o guia `docs/hospedagem-hostinger.md`. Verificar numa cópia limpa: `npm ci`, build e `npm start`.
2. **Login (antes de qualquer dado novo no ar):** sessão por cookie seguro, senha com hash. Perfis: `dono` (tudo) e `nutricionista` (somente leitura em Alimentação, Corpo e Treinos). Autorização checada no servidor em cada página e ação, e não só no menu.
3. **Esquema MySQL de Alimentação:** alimentos, consumos, itens de consumo, água, metas, conversões de unidades, plano e opções; mais `usuarios`, `convites` e `log_acessos`. Corpo e Treinos entram depois, mesmo desenho.
4. **Camada de dados:** uma interface de repositório com duas implementações (planilha e MySQL), trocada por módulo. Alimentação passa para MySQL; os outros módulos seguem na planilha até a vez deles.
5. **Importação:** script que lê a planilha (já com backup) e grava no MySQL, rodado por você no servidor. Idempotente, com contagem de linhas de origem e destino.
6. **Tela do nutricionista:** protótipo com prints antes de implementar.
7. **Virada de Alimentação:** comparar totais (kcal, água, últimos 30 dias) entre planilha e MySQL, congelar a planilha e trocar a chave do módulo.
8. Repetir 3 a 7 para Corpo e Treinos.

## Acesso do nutricionista
- Convite por link, ele define a própria senha; você pode revogar a qualquer momento.
- Vê só Alimentação, Corpo e Treinos, em leitura. Observações dele, se você quiser, ficam em tabela separada.
- Todo acesso fica no log. Dados de saúde são sensíveis (LGPD): registrar o consentimento.

## Pendências suas
- Criar o banco MySQL no hPanel (nome, usuário e senha; o host é `localhost` quando o app roda na própria Hostinger).
- Rodar a importação no servidor.
- Prints de `Consumos.gs`, `Api.gs`, `Auditoria.gs` e do arquivo das opções de Nutrição.

## Riscos
- O app na Hostinger e o MySQL precisam estar na mesma conta; a Netlify não alcança o MySQL da Hostinger.
- Se o login (etapa 2) não entrar antes do nutricionista, os dados de saúde ficam expostos. Por isso ele vem primeiro.
