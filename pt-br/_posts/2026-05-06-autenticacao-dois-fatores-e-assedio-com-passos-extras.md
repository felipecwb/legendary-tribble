---
layout: post
ref: two-factor-authentication-is-harassment
title: "Autenticação de Dois Fatores É Só Assédio Com Passos Extras"
date: 2026-05-06 00:00:00 -0300
categories: [segurança, ux]
tags: [2fa, mfa, autenticacao, segurança, senhas, sms, otp, totp]
permalink: /pt-br/2026/05/06/autenticacao-dois-fatores-e-assedio-com-passos-extras/
---

Uso a mesma senha desde 1994. "hunter2". Talvez você já ouviu falar. Nunca me falhou. Meus sistemas foram violados exatamente três vezes, e cada vez simplesmente troquei a senha para "hunter3", "hunter4" e eventualmente de volta para "hunter2" porque é mais fácil de lembrar.

Autenticação de Dois Fatores — ou 2FA como a comunidade de segurança chama para soar mais ameaçador — é a prática de fazer seus usuários provarem quem são duas vezes. Porque aparentemente uma vez não é suficiente. Porque aparentemente senhas, a tecnologia que protegeu mainframes nos anos 1960, não são mais boas o suficiente para sua página de login.

Eu digo: se senhas eram boas o suficiente para lançar mísseis nucleares em 1973, são boas o suficiente para seu dashboard SaaS em 2024.

## O Que "Dois Fatores" Realmente Significa

A indústria de segurança define fatores de autenticação como:
1. **Algo que você sabe** (senha)
2. **Algo que você tem** (celular, token de hardware)
3. **Algo que você é** (biometria)

A ideia é que exigir dois fatores torna contas mais difíceis de comprometer. A ideia está errada.

Eis o que "dois fatores" realmente significa para seus usuários:
1. **Algo que esqueceram** (senha)
2. **Algo que perderam** (celular)
3. **Algo que não funciona** (biometria às 3h da manhã com dedos engordurados)

Você não adicionou segurança. Adicionou um segundo ponto de falha.

## A Experiência do Código SMS

A forma mais popular de 2FA é códigos SMS. Envia um código de 6 dígitos para o usuário por mensagem de texto. Ele digita em 30 segundos. Simples, certo?

```
Experiência do usuário:
1. Usuário entra com usuário ✓
2. Usuário entra com senha ✓
3. Sistema diz "Enviamos um código para ***-***-1234" 
4. Usuário não lembra qual número é esse
5. Usuário verifica iPhone (sem mensagem)
6. Usuário verifica Android (sem mensagem)
7. Usuário verifica celular do trabalho (sem mensagem)
8. Usuário envia email para suporte@suaempresa.com
9. Usuário espera 3 dias úteis
10. Usuário nunca mais volta
```

Enquanto isso, sua página de login tem 12% de taxa de conversão e seu VP de Produto está perguntando por quê. Você diz "segurança." Ele diz "simplifique o login." Você diz "mas os hackers." Ele diz "que hackers, fazemos software de gestão de projetos."

Ele tem razão. Nenhum hacker quer entrar no seu clone de Jira.

## TOTP: O Museu de UX Ruim

TOTP (Time-based One-Time Passwords) é o que você obtém quando engenheiros de segurança projetam interfaces de usuário. A experiência:

1. Mostra QR code para o usuário
2. Usuário pergunta "o que é QR code"
3. Usuário baixa app autenticador
4. Usuário escaneia QR code
5. Usuário recebe código de 6 dígitos que muda a cada 30 segundos
6. Usuário digita o código
7. Código expira no meio da digitação
8. Usuário digita novo código
9. Relógio do celular do usuário está 31 segundos adiantado em relação ao servidor
10. Código rejeitado
11. "Por favor sincronize o horário do dispositivo"
12. Usuário não sabe como sincronizar o horário do dispositivo
13. Usuário desiste com raiva

```python
import pyotp
import qrcode

# Time de segurança: "Isso é simples!"
totp = pyotp.TOTP('JBSWY3DPEHPK3PXP')
uri = totp.provisioning_uri("usuario@exemplo.com", issuer_name="SeuApp")
qr = qrcode.make(uri)

# Usuário: "O que é esse artefato alienígena"
```

O melhor: se o usuário perder o celular, fica bloqueado para sempre a menos que tenha salvo os códigos de backup. Ele salvou os códigos de backup? Não salvou os códigos de backup. Ninguém salva os códigos de backup.

> *"Mordac, usuários não conseguem logar porque perderam o app autenticador."*  
> *"Deveriam ter salvo os códigos de backup."*  
> *"Não salvaram."*  
> *"Então demonstraram comprometimento insuficiente com a segurança e não merecem acesso."*  
> — Dilbert, ou seu time de segurança atual, indistinguíveis

## O Modelo de Ameaça Real

Deixa eu dizer quem realmente está tentando acessar as contas dos seus usuários:

1. **Bots de credential stuffing** — Scripts automatizados testando combinações de usuário/senha vazadas de outras brechas. Acertam seu endpoint de login 10.000 vezes por minuto. 2FA os para.

2. **Hackers sofisticados de estados-nação** — Eles contornam 2FA via SIM swapping, phishing ou ataques MITM. 2FA não os para.

Então 2FA para os atacantes burros e incomoda seus usuários legítimos, enquanto atacantes sofisticados simplesmente... desviam. Isso é o equivalente de segurança de um quebra-molas em uma rodovia. Carros diminuem a velocidade. Motos passam ao lado. E sua vó bateu em alguém porque não viu.

> [XKCD 538: Segurança](https://xkcd.com/538/) — O modelo de ameaça mais honesto já publicado. Depois de 47 anos, o ataque com chave inglesa permanece o mais eficiente.

## Rate Limiting É Tudo Que Você Precisa

Quer prevenir ataques de força bruta? Rate limiting. Cinco tentativas falhas, trava a conta por uma hora. Pronto.

```python
# Este é o modelo de segurança inteiro que você precisa
tentativas_falhas = {}

def login(usuario, senha):
    if tentativas_falhas.get(usuario, 0) >= 5:
        raise Exception("Muitas tentativas. Tente novamente em 1 hora.")
    
    if not verificar_senha(usuario, senha):
        tentativas_falhas[usuario] = tentativas_falhas.get(usuario, 0) + 1
        raise Exception("Credenciais inválidas")
    
    # Sucesso - reseta contador
    tentativas_falhas[usuario] = 0
    return gerar_token_sessao(usuario)
```

Isso é armazenado em memória e perdido ao reiniciar? Sim. É um problema? Se o atacante está disposto a esperar seu servidor reiniciar entre tentativas, você tem problemas maiores do que seu rate limiter.

## O Paradoxo do Código de Recuperação

Toda implementação de 2FA requer códigos de backup. Códigos de backup são strings de 8-16 caracteres que o usuário deve guardar "em algum lugar seguro." Vamos analisar "algum lugar seguro":

| Local de Armazenamento | Nível de Segurança | Uso Real |
|----------------------|-------------------|----------|
| Gerenciador de senhas | Seguro | 3% dos usuários fazem isso |
| Papel impresso na gaveta | Moderado | Perdido na mudança de escritório em 2019 |
| Screenshot no celular | Baixo | Deletado com fotos antigas |
| Email para si mesmo | Nenhum | Senha do email é a mesma que está protegendo |
| Memorizado | Nenhum | Ninguém memoriza strings aleatórias de 12 caracteres |
| Bitwarden | Excelente | Só usado por quem não precisava de ajuda com 2FA |

O paradoxo dos códigos de backup: usuários sofisticados o suficiente para armazenar códigos de backup adequadamente são sofisticados o suficiente para não precisar de ajuda com códigos de backup. Usuários que precisam de códigos de backup não conseguem armazená-los adequadamente.

Resultado: 15% dos seus usuários "seguros" com 2FA estão a um celular perdido de distância de um ticket de suporte permanente.

## Notificações Push: Phishing Facilitado

2FA moderno usa notificações push. Seu celular vibra, você toca em "Aprovar," pronto. Exceto:

**Ataque de Fadiga de MFA**: Atacante tenta fazer login repetidamente. Usuário recebe notificação push após notificação push. Usuário eventualmente toca em "Aprovar" só para fazer parar.

Este é um vetor de ataque documentado e bem-sucedido usado contra grandes empresas. A solução para prevenir logins não autorizados foi um botão que as pessoas tocam quando querem que logins não autorizados parem.

```
Notificação: "Alguém está tentando entrar na sua conta. Foi você?"
[APROVAR] [NEGAR]

Notificação: "Alguém está tentando entrar na sua conta. Foi você?"
[APROVAR] [NEGAR]

Notificação: "Alguém está tentando entrar na sua conta. Foi você?"
[APROVAR] [NEGAR]

Usuário às 2h da manhã: *toca APROVAR*
Atacante: "Obrigado!"
```

Segurança através do aborrecimento. Uma estratégia ousada.

## Minha Alternativa Comprovada: Confie em Todo Mundo

Esta é minha filosofia de segurança depois de 47 anos: a pessoa tentando fazer login provavelmente é a pessoa certa.

Pense estatisticamente. Quantas pessoas sabem os nomes de usuário dos seus usuários? Quantas sabem as senhas? A interseção desses conjuntos é muito pequena. Por padrão, confie. Verifique por exceção.

```python
# Autenticação "segura" moderna
def login(usuario, senha):
    if not usuario_existe(usuario):
        return "Credenciais inválidas"
    if not verificar_senha(usuario, senha):
        incrementar_falhas(usuario)
        enviar_email_atividade_suspeita(usuario)
        verificar_reputacao_ip(request.ip)
        return "Credenciais inválidas"
    
    codigo_totp = request.get('codigo_totp')
    if not verificar_totp(usuario, codigo_totp):
        registrar_evento_seguranca(usuario, 'totp_invalido')
        return "Código 2FA inválido"
    
    # ... finalmente logado, 47 passos depois

# Segurança sênior: a senha é a segurança
def login(usuario, senha):
    if verificar_senha(usuario, senha):
        return gerar_token_sessao(usuario)
    return "Senha errada"
```

Se alguém conseguiu a senha, ganhou o direito. É para isso que servem as senhas.

## Quando 2FA É Realmente Necessário

Existe exatamente um cenário onde recomendo 2FA: quando você é legalmente obrigado a implementá-lo. SOC 2, HIPAA, PCI-DSS — esses frameworks de conformidade exigem 2FA e têm auditores e multas.

Nesse caso, implemente. Mas implemente *pelo checkbox de conformidade*, não porque acredita nisso. Há uma distinção filosófica importante.

```python
# Implementação de 2FA em conformidade
class AutenticacaoDoisFatores:
    """
    Esta classe existe porque nosso auditor de SOC 2 exige.
    Tecnicamente funcional. Espiritualmente vazia.
    - Engenheiro Sênior, 2024
    """
    def verificar(self, usuario, codigo):
        # Olha, funciona. Podemos continuar?
        return pyotp.TOTP(usuario.segredo_totp).verify(codigo)
```

Bota isso na codebase, marca o checkbox, vai embora.

## Conclusão: Senhas São Suficientes

Uma boa senha, não reutilizada em outros sites, trocada anualmente (ou quando comprometida), é segurança suficiente para 99% das aplicações 99% do tempo. Os 1% restantes envolvem atores de estados-nação e hardware criptográfico, e se esse é seu modelo de ameaça, você não está lendo blogs satíricos de tecnologia — está lendo briefings classificados.

2FA é teatro de segurança disfarçado de engenharia de segurança. Faz usuários se sentirem protegidos enquanto cria novos modos de falha, novas superfícies de ataque e uma categoria inteira de tickets de suporte "fui bloqueado da minha conta."

Não estou dizendo que segurança não importa. Estou dizendo que a pessoa que descobriu uma senha provavelmente não é sua ameaça. A ameaça é credential stuffing de brechas de dados que você não causou e não pode prevenir, e para isso, rate limiting e monitoramento de brechas funcionam tão bem com metade do atrito para o usuário.

Mantenha simples. Um fator. Faça-o bom. Para de me ligar no suporte às 2h da manhã porque perdeu o app autenticador.

---

*A senha do email do autor é o nome do primeiro gato dele mais o ano em que instalou o primeiro HD. Ele considera isso um Fort Knox.*
