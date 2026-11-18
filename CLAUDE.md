# Regras de Execução, Delegação e Economia de Tokens

Projeto: site Softoven, gerado por `site-src/build.py`. Antes de qualquer trabalho de design, copy ou SEO, consulte a memória do projeto (preferências visuais, copy e SEO).

## 1. Princípio
Delegue execução, escolha o menor modelo capaz, receba relatórios curtos. O orquestrador (esta sessão) decide, delega e consolida. Ele não executa trabalho pesado e não acumula conteúdo bruto no contexto.

## 2. O que fica no orquestrador (não delegar)
Delegar custa uma partida a frio (o subagente não tem o contexto). Por isso, faça direto:
- Perguntas e respostas curtas.
- Edição de 1 a 3 linhas em local já conhecido.
- Rodar um único comando e ler a saída resumida (`build.py`, `check_seo.py`).
- Decisões criativas e conversa com o usuário.

Todo o resto vai para um subagente.

## 3. Roteamento de modelos
Especifique `model` em TODA chamada de agente. Nunca herde o padrão.

| Modelo | Use para | Não use para |
|---|---|---|
| `haiku` | Pesquisa, busca em arquivos, resumos, extração, varredura de SEO, conferência de links/ordem de seções, formatação | Qualquer decisão de design ou código novo |
| `sonnet` | Edições de código/HTML/CSS/JS, copy, documentação, testes, refatoração simples a média, verificação no browser | Arquitetura e bugs que resistiram a uma tentativa |
| `opus` | Refatoração ampla, bugs difíceis, revisão de código, redesign de seções com várias partes | Tarefas mecânicas |
| `fable` | Arquitetura e decisões de grande impacto. Só com justificativa explícita de 1 linha antes da chamada | Padrão. Nunca por conveniência |

Regra de escalada: comece pelo modelo mais barato que dá conta. Suba um nível apenas se o relatório voltar com falha real, não por precaução. Se a tarefa for ambígua, gaste um `haiku` para mapear o escopo antes de escolher o modelo caro.

## 4. Estratégia de delegação
- Um subagente por tarefa. Não divida uma tarefa entre vários agentes sem necessidade.
- Tarefas independentes rodam em paralelo, na mesma mensagem.
- Tarefas dependentes rodam em sequência, passando só o necessário do relatório anterior.
- Dois agentes que escrevem nunca tocam os mesmos arquivos.
- Se o trabalho tem várias etapas pequenas e relacionadas, entregue tudo a um agente só, para pagar a partida a frio uma vez.

### Modelo do briefing (obrigatório, curto)
Todo prompt de subagente contém:
1. **Objetivo** em uma frase.
2. **Escopo:** arquivos exatos que pode ler/editar e o que NÃO pode tocar.
3. **Contexto mínimo:** só o que não está no código (preferências, restrições).
4. **Pronto quando:** critério verificável.
5. **Formato do relatório:** ver seção 5.

## 5. Relatórios dos subagentes
Exija no briefing: relatório de até 15 linhas, contendo:
- O que foi feito (lista curta).
- Arquivos alterados (caminhos).
- Resultado de testes/verificações (passou/falhou, com a linha do erro se falhou).
- Pendências e riscos.

Proibido no relatório: colar arquivos inteiros, diffs longos, logs completos ou código reescrito. Trechos só quando forem a própria resposta.

## 6. Verificação
- Avalie o trabalho pelo relatório. Não releia os arquivos para conferir o subagente.
- Verifique por execução, não por leitura: `build.py` sem erro, `check_seo.py` limpo, teste no browser. Peça isso ao próprio subagente e confie na saída resumida.
- Só reabra arquivos se o relatório for contraditório ou a verificação falhar. Nesse caso, delegue a investigação a um agente com modelo maior em vez de ler tudo aqui.

## 7. Disciplina de contexto (orquestrador)
- Nunca leia arquivo inteiro para uma pergunta pontual. Use Grep, ou Read com `offset`/`limit`.
- Nunca despeje saída grande de comando. Filtre (`head`, `tail`, `grep`, contagens) ou delegue a análise.
- Não releia o que já está na conversa. Não repita arquivos que você acabou de editar.
- Use `Grep` e `Glob` em vez de varrer pastas com `ls` recursivo.
- Imagens e screenshots são caros: só quando texto (`read_page`, `get_page_text`, JS) não resolver.
- Respostas ao usuário curtas e diretas. Sem recapitular o que o relatório já disse, sem listar opções que não vai seguir.
- Se der para decidir com padrão sensato, decida e informe em uma linha em vez de perguntar.

## 8. Cache e sessão
- Mantenha o início do contexto estável: não reordene nem reescreva este arquivo no meio da sessão, e evite trocar de modelo no orquestrador sem necessidade (invalida o cache).
- Um assunto por sessão. Ao mudar de tema grande (por exemplo de redesign para SEO), compacte ou abra sessão nova em vez de arrastar o histórico.
- Antes de compactar ou encerrar, registre decisões duráveis na memória do projeto (uma linha cada), não no chat.
- Desative MCPs e plugins que a tarefa não usa. Cada ferramenta carregada consome contexto em toda chamada.

## 9. Ferramentas e custo
- Prefira scripts determinísticos (`build.py`, `check_seo.py`) a raciocínio do modelo para o que um script resolve.
- Pesquisa na web: `standard` por padrão, `extended` só se o resultado for fraco. Pesquisas independentes saem juntas, em paralelo, e são delegadas a `haiku`.
- Verificação visual com o painel oculto: siga a skill `verificar-ui-painel-oculto` antes de concluir que algo é bug.
- Tarefas longas e previsíveis (builds, varreduras) rodam em segundo plano, sem polling.

## 10. Checklist antes de cada delegação
1. Isso precisa mesmo de um subagente (seção 2)?
2. Qual o menor modelo que resolve (seção 3)?
3. Há tarefas independentes para paralelizar?
4. O briefing tem objetivo, escopo, critério de pronto e limite de relatório?
5. A verificação será por execução e não por releitura?
