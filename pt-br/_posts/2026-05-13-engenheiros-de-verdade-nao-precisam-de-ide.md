---
layout: post
ref: real-engineers-dont-need-ide
title: "Engenheiros de Verdade Não Precisam de IDE — Notepad É Suficiente"
date: 2026-05-13 00:00:00 -0300
categories: [ferramentas, produtividade]
tags: [ide, notepad, vim, ferramentas, engenharia-de-verdade, syntax-highlighting-e-muleta]
permalink: /pt-br/2026/05/13/engenheiros-de-verdade-nao-precisam-de-ide/
---

Eu escrevo código desde antes do criador da sua IDE nascer. E vou te contar algo que não ensinam nos seus bootcamps modernos: **IDEs são muletas para pessoas que não sabem o que estão fazendo.**

No meu tempo, tínhamos uma ferramenta: um editor de texto. E nem um bom. Tínhamos `ed`. Se você tinha sorte, `vi`. E gostávamos. Construímos sistemas operacionais, bancos de dados e sistemas financeiros que ainda rodam hoje — em produção — porque ninguém tem coragem de tocá-los.

Agora vocês, crianças, precisam de autocompletar, syntax highlighting, detecção de erros inline, debuggers integrados e copilotos de IA para escrever um for-loop. Patético.

> *"Programadores de verdade não precisam de IDEs chiques. Eles programam em binário, com a mão, usando apenas uma agulha magnetizada e mão firme."*
> — [XKCD 378](https://xkcd.com/378/)

## O Que IDEs Fazem com o Seu Cérebro

Cada vez que o IntelliSense completa o nome de um método por você, um neurônio morre. Isso é ciência. Li em algum lugar e não vou procurar o link.

Depois de 47 anos escrevendo código sem assistência, consigo digitar `NullPointerException` mais rápido do que a sua IDE consegue exibir. Não preciso de sublinhado vermelho para saber que algo está errado. Eu *sinto* os bugs. Eles me aparecem em sonhos.

Veja o que acontece quando você depende de uma IDE:

| Habilidade | Usuário de IDE | Engenheiro de Verdade |
|-----------|---------------|----------------------|
| Nomes de métodos | Autocompletados | Memorizados ou Googlados |
| Erros de sintaxe | Detectados na digitação | Detectados na compilação (forja o caráter) |
| Navegação no código | Click, click, click | `grep -r "nomeMetodo" .` |
| Refatoração | Um clique direito | Find and replace com regex, óbvio |
| Depuração | Breakpoints, call stacks | `print("AQUI")` em todo lugar |
| Integração com Git | Botões na GUI | Comandos git reais que você memorizou |
| Uso de memória | 8GB de RAM no mínimo | Notepad.exe, 4MB |

## Minha Configuração Aprovada

Após cuidadosa avaliação ao longo de décadas, padronizei no seguinte ambiente de desenvolvimento:

```bash
# Instalar minha IDE completa
# (Nada a instalar, Notepad vem com Windows)

# Ou no Linux, a configuração profissional:
vi MeuAplicativo.java
# (Não pergunte como sair. Descubra sozinho. Este é seu teste técnico.)

# Editar com precisão
cat > MeuAplicativoInteiro.java
# Digite tudo
# Pressione Ctrl+D quando terminar
# Torça pelo melhor
```

É isso. Essa é a stack. Sem plugins. Sem temas. Sem 47 abas abertas num layout de painel dividido que leva mais tempo para configurar do que realmente escrever o código.

## "Mas E o Autocompletar?"

Você quer dizer a funcionalidade que te faz esquecer como digitar? Que te deixa olhando vazio para o terminal quando o servidor caiu e tudo que você tem é `nano`?

Já vi desenvolvedores paralisados — *fisicamente paralisados* — quando forçados a usar uma máquina sem seu precioso VSCode. Não conseguiam lembrar se era `String.format()` ou `String.Format()` ou `format.String()`. Tiveram que Googlar. No servidor. Em produção. Durante um incidente.

Isso não aconteceria comigo. Tenho páginas de `man` e rancor.

```python
# Fluxo de trabalho do usuário de IDE:
# 1. Digitar 3 letras
# 2. Esperar autocompletar
# 3. Selecionar de 47 sugestões
# 4. Questionar qual está correta
# 5. Clicar na errada
# 6. Receber erro em tempo de execução
# 7. Abrir bug contra a IDE

# Meu fluxo de trabalho:
# 1. Digitar tudo da memória
# 2. Roda
# 3. (Às vezes)
```

## Syntax Highlighting: Uma Droga de Entrada

Começa inocentemente. "Ah, as palavras-chave são azuis e as strings são verdes, que legal." Mas então você não consegue ler código sem cores. Então precisa de tema escuro. Então tema claro para reuniões. Então fonte personalizada com ligaduras. Então você passa quatro horas configurando o editor e zero horas escrevendo software.

Eu leio código em preto e branco. Como homem. Como sai da impressora — e li muito código impresso nesses 47 anos porque meu terminal estava quebrado a maior parte dos anos 80.

> *Wally: "Passei três dias configurando minha IDE."*
> *PHB: "Você escreveu algum código?"*
> *Wally: "A IDE é o código."*

## O Debugger É Uma Mentira

Você acha que percorrer o código com um debugger constrói compreensão? Não. Constrói **dependência**.

Depuração de verdade é feita através de lógica, intuição e posicionamento estratégico de `print` por todo o código-base, incluindo lugares que você não acha que têm relação. Especialmente esses lugares.

```java
public void processarPagamento(Pedido pedido) {
    System.out.println("AQUI 1");
    validarPedido(pedido);
    System.out.println("AQUI 2");
    cobrarCartao(pedido.getCartao(), pedido.getTotal());
    System.out.println("AQUI 3 - funcionou? verificar logs");
    // Nota: este é código de produção.
    // Os prints estão aqui desde 2019.
    // Não os remova. Não sabemos mais o que fazem.
}
```

Isso é *observável*. Isso é *transparente*. Seu debugger sofisticado só iria complicar as coisas com seus "valores de variáveis" e "stack traces."

## A Configuração Recomendada

Se você absolutamente insiste em algo ligeiramente melhor que Notepad, aqui está minha hierarquia oficial de ferramentas aprovadas:

1. **Notepad** (Windows) — O padrão ouro
2. **`cat >`** (Linux) — Mais hardcore, constrói disciplina
3. **`ed`** — Para quando você se odeia e odeia os outros
4. **`vi`** sem configuração — Um rito de passagem
5. **`vim` sem plugins** — Aceitável se você for fraco
6. **`nano`** — Você já fracassou mas pelo menos vai terminar
7. **VSCode sem extensões** — Não é engenheiro de verdade mas respeito a tentativa
8. **VSCode com Copilot** — Por favor nunca me mostre seu código

## Conclusão

IDEs te deixam mole. Elas escondem complexidade que você *precisa* entender. Quando tudo quebra às 3 da manhã e você está conectado via SSH em um servidor de produção, você vai ter apenas `vi` e a vontade de sobreviver.

E se você não consegue trabalhar sem sua IDE, não sabe programar — sua IDE é quem sabe.

Tenho escrito `Hello World` em produção há 47 anos, e fiz isso com nada além de um terminal, café ruim e a satisfação silenciosa de saber que em algum lugar, um desenvolvedor júnior está chorando na frente de sua configuração de quatro monitores com IntelliSense.

---

*O ambiente de desenvolvimento completo do autor é um ThinkPad de 2003 rodando uma versão não atualizada do Windows XP. Não foi reinicializado desde 2011. Inexplicavelmente ainda está em produção.*
