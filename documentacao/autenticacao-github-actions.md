# Autenticação entre repositórios no GitHub Actions

Como autorizar o workflow de um repositório a **escrever** em outro.

No nosso caso, o repositório de código
[`ronidomingues/git-ia-docs`](https://github.com/ronidomingues/git-ia-docs)
compila os PDFs e precisa enviá-los para
[`andradasdev/git-ia`](https://github.com/andradasdev/git-ia), que os publica.

---

## Por que o token padrão não serve

Todo workflow recebe automaticamente um token chamado `GITHUB_TOKEN`. Ele é
prático, mas tem **duas limitações** que inviabilizam o nosso caso:

| Limitação | Consequência aqui |
|---|---|
| Só tem permissão no **próprio repositório** em que o workflow roda | O `GITHUB_TOKEN` de `git-ia-docs` **não escreve** em `andradasdev/git-ia` |
| Pushes feitos com ele **não disparam outros workflows** | Mesmo que escrevesse, o workflow de Pages do `git-ia` **não seria acionado** |

A segunda limitação existe de propósito, para evitar laços infinitos de
workflows disparando uns aos outros. É por isso que precisamos de um token de
usuário (PAT): **um push feito com PAT dispara workflows normalmente**, e é
disso que dependemos para o Pages publicar.

---

## O que vamos criar

```
┌─────────────────────────────────────┐
│  ronidomingues/git-ia-docs          │   repositório de CÓDIGO
│                                     │
│  Settings > Secrets and variables   │
│    > Actions                        │
│      GIT_IA_ANDRADASDEV  ← o valor  │
└──────────────┬──────────────────────┘
               │  git push autenticado com o token
               v
┌─────────────────────────────────────┐
│  andradasdev/git-ia                 │   repositório de PUBLICAÇÃO
│                                     │
│  recebe: apostila.pdf,              │
│          apresentacao.pdf, card.png │
│  dispara: workflow de Pages         │
└─────────────────────────────────────┘
```

---

## Passo 1 — Criar o token fine-grained

Um **PAT fine-grained** (*Personal Access Token* granular) é preferível ao
clássico porque você escolhe **exatamente** qual repositório ele alcança e
**exatamente** o que ele pode fazer lá.

1. Acesse <https://github.com/settings/personal-access-tokens/new>
   (ou: foto do perfil → **Settings** → **Developer settings** →
   **Personal access tokens** → **Fine-grained tokens** → **Generate new token**).

2. Preencha:

   | Campo | Valor |
   |---|---|
   | **Token name** | `publicar-em-andradasdev-git-ia` |
   | **Description** | Envio de artefatos compilados do git-ia-docs |
   | **Resource owner** | **`andradasdev`**, e não a sua conta pessoal |
   | **Expiration** | 90 dias, ou o menor prazo que você aceite renovar |

   > ⚠️ **`Resource owner` é o campo que mais gera erro.** Com a conta
   > pessoal nesse campo, o token **não enxerga** os repositórios da
   > organização, e o push falha com `403` mesmo com todas as permissões
   > marcadas.

3. Em **Repository access**, escolha **Only select repositories** e selecione
   apenas **`git-ia`**.

4. Em **Repository permissions**, encontre **Contents** e marque
   **Read and write**. É a **única** permissão necessária. A permissão
   **Metadata: Read-only** é marcada sozinha pelo GitHub, e isso é normal.

5. Clique em **Generate token** e **copie o valor na hora**. Ele começa com
   `github_pat_` e **não aparece de novo**.

> **Dá para reaproveitar o token do `sql`?** Dá, se você editar o token que já
> existe e acrescentar `git-ia` em **Repository access**. Mas um token por
> repositório limita o estrago se um deles vazar, e é o que recomendamos.

---

## Passo 2 — Conferir a política de PATs da organização

`andradasdev` é uma **organização**. Os PATs fine-grained já foram liberados
nela para o repositório `sql`, então em geral não há nada a fazer. Se a
organização exigir aprovação, aprove o token novo em
<https://github.com/organizations/andradasdev/settings/personal-access-token-requests>.

> Enquanto estiver pendente, o token existe mas **não funciona**: o push falha
> com `403`, sem explicar o motivo.

---

## Passo 3 — Guardar o token como secret

O token vai para o repositório **de código**, que é quem precisa dele.

1. Acesse <https://github.com/ronidomingues/git-ia-docs/settings/secrets/actions>.
2. **New repository secret**:

   | Campo | Valor |
   |---|---|
   | **Name** | `GIT_IA_ANDRADASDEV` |
   | **Secret** | o valor `github_pat_...` copiado no Passo 1 |

Pelo terminal dá no mesmo, e o valor é pedido sem aparecer na tela:

```bash
gh secret set GIT_IA_ANDRADASDEV -R ronidomingues/git-ia-docs
```

> O nome precisa ser exatamente `GIT_IA_ANDRADASDEV`, que é o que
> `.github/workflows/build.yml` procura. Depois de salvo, o valor não pode ser
> lido por ninguém, só substituído, e o GitHub o mascara nos logs.

---

## Passo 4 — Como o workflow usa o token

O trecho relevante de `.github/workflows/build.yml`, no repositório de código:

```yaml
- name: Enviar artefatos para andradasdev/git-ia
  env:
    PUBLISH_TOKEN: ${{ secrets.GIT_IA_ANDRADASDEV }}
  run: |
    git clone --depth 1 \
      "https://x-access-token:${PUBLISH_TOKEN}@github.com/andradasdev/git-ia.git" \
      /tmp/publicacao
    # ... copia os artefatos ...
    cd /tmp/publicacao
    git push origin HEAD:main
```

Três detalhes que valem entender:

- **`x-access-token`** é o nome de usuário convencionado pelo GitHub para
  autenticar com token via HTTPS. Quem autentica de fato é o token.
- **O secret entra por `env:`, nunca interpolado no corpo do script.**
  Escrever `${{ secrets.GIT_IA_ANDRADASDEV }}` dentro do `run:` faz o valor
  virar texto do comando.
- **Nunca use `set -x`** num passo que manipula o token: o modo de rastreamento
  imprime cada comando expandido, com a URL e o token dentro.

---

## Passo 5 — Pages no repositório de publicação

Em <https://github.com/andradasdev/git-ia/settings/pages>, **Source** precisa
estar em **GitHub Actions** (e não em *Deploy from a branch*). Neste
repositório isso já foi configurado na criação.

---

## Passo 6 — Testar

No repositório de código: **Actions** → **Compilar LaTeX e publicar em
andradasdev/git-ia** → **Run workflow**. Ou pelo terminal:

```bash
gh workflow run build.yml -R ronidomingues/git-ia-docs
```

Fluxo esperado:

1. o workflow do código compila, verifica e commita os PDFs;
2. envia os artefatos para `andradasdev/git-ia`;
3. o push dispara o workflow de Pages lá;
4. o site sai em <https://andradasdev.github.io/git-ia/>.

---

## Renovação

O token **expira** na data escolhida no Passo 1. A partir daí o push falha com
`403`, e o site para de ser atualizado sem aviso para quem só olha o site.
Para renovar, use **Regenerate** na página do token (preserva nome e
permissões) e repita o Passo 3.

---

## Diagnóstico de erros

| Sintoma | Causa provável | Correção |
|---|---|---|
| `O secret GIT_IA_ANDRADASDEV não está configurado` | O secret não existe ou o nome está diferente | Passo 3 |
| `403` logo depois de criar o token | Token pendente de aprovação na organização | Passo 2 |
| `403` com o token aprovado | `Resource owner` ficou como a conta pessoal, ou faltou **Contents: Read and write** | Recrie ou edite o token (Passo 1) |
| `Repository not found` | O token não inclui o repositório `git-ia` | **Repository access** → selecione `git-ia` |
| Push funciona, mas o Pages não publica | Source do Pages não está em *GitHub Actions* | Passo 5 |
| Tudo verde, site com PDF antigo | Os artefatos eram idênticos aos publicados | Comportamento esperado |
| `403` que começou do nada | O token expirou | Seção **Renovação** |

---

## Alternativas

O PAT não é a única forma:

- **Deploy Key (SSH):** a chave pública vira *Deploy Key* com escrita em
  `andradasdev/git-ia`, e a privada vira secret no repositório de código. Não
  expira e alcança um único repositório, mas exige configurar SSH no runner.
- **GitHub App:** gera tokens de curta duração sob demanda. É o mais robusto
  para muitos repositórios e o mais trabalhoso para apenas dois.
- **`repository_dispatch`:** em vez de escrever no outro repositório, o
  primeiro só o avisa, e ele busca os artefatos por conta própria.
