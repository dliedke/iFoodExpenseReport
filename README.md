# iFood Expense Report

Script em Python que faz login no [iFood](https://www.ifood.com.br/pedidos),
coleta seus pedidos recentes e gera uma planilha de despesas estilizada
(`pedidos_ifood_30dias.xlsx`) com uma linha por pedido, mais total, média e
contagem. É uma automação de navegador via Selenium, **interativa por design**:
você loga manualmente uma vez e o script faz o resto.

## Pré-requisitos

- **Python 3.9+**
- **Microsoft Edge** instalado (o driver é baixado automaticamente pelo
  Selenium Manager — você não precisa instalar nada à mão).

## Instalação

```bash
pip install -r requirements.txt
```

## Como rodar

```bash
python iFoodExpenseReport.py
```

O que acontece:

1. Abre o Edge na página de pedidos do iFood.
2. **Faça o login manualmente** no navegador que abriu (usuário/senha,
   verificação, Cloudflare etc.). Quando terminar, volte ao terminal e
   pressione **ENTER**.
3. O script rola a lista, coleta os links dos pedidos e visita cada um para
   extrair **restaurante, valor e data**.
4. Ao final, gera o arquivo `pedidos_ifood_30dias.xlsx` na pasta do projeto e,
   no Windows, abre a planilha automaticamente.

> Na primeira execução você precisa logar. Nas próximas, a sessão costuma ser
> reaproveitada (veja [Perfil persistente](#perfil-persistente)), então
> normalmente basta apertar ENTER.

## Configuração

Os parâmetros ficam no topo de `iFoodExpenseReport.py`:

| Constante     | Padrão | O que faz                                                            |
|---------------|--------|----------------------------------------------------------------------|
| `DIAS`        | `30`   | Janela de tempo: pedidos mais antigos que isso são descartados.      |
| `MAX_PEDIDOS` | `62`   | Teto de pedidos coletados, para não varrer histórico antigo demais.  |

A coleta para no que vier primeiro: ao atingir `MAX_PEDIDOS` **ou** ao
ultrapassar a janela de `DIAS` (os pedidos vêm do mais novo para o mais antigo).

## Perfil persistente

O script usa um perfil de Edge dedicado na pasta `edge_profile/`, então a
sessão de login do iFood e a liberação do Cloudflare **sobrevivem entre as
execuções** — por isso você geralmente só loga uma vez.

## Cache

Os dados extraídos de cada pedido são memoizados em `cache_pedidos.json`
(chaveado pelo GUID do pedido) e salvos a cada novo pedido. Re-execuções pulam
pedidos já coletados.

Se um pedido vier com dado errado/desatualizado, **remova a entrada
correspondente de `cache_pedidos.json`** (ou apague o arquivo inteiro) para
forçar a re-coleta.

## Artefatos gerados (não versionados)

`edge_profile/`, `cache_pedidos.json` e os arquivos `.xlsx` são saídas de
execução e estão no `.gitignore`.

## Observações

- Código e textos da interface estão em português do Brasil.
- Valores são interpretados no formato brasileiro (`R$ 66,10` → `66.10`;
  datas em `dd/mm/aaaa`).
- Não há testes, linter ou build. Os arquivos `.pyproj`/`.slnx` servem apenas
  para abrir o projeto no Visual Studio; o script roda standalone com `python`.
