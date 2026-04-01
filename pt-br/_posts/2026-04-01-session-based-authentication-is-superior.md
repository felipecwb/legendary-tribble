---
layout: post
ref: session-based-authentication-is-superior
title: "Autenticação Baseada em Sessão É Superior: Por Que JWTs São Uma Conspiração"
date: 2026-04-01 00:00:00 -0300
categories: [seguranca, backend]
tags: [autenticacao, sessoes, jwt, php, cookies, seguranca, conspiracao]
permalink: /pt-br/:year/:month/:day/session-based-authentication-is-superior/
---

Depois de 47 anos assistindo tendências de autenticação irem e virem, eu vi a indústria inteira ser enganada pelos JSON Web Tokens. Deixa eu te contar sobre a era dourada das **sessões PHP** e por que devemos retornar a ela imediatamente.

## Os Bons e Velhos Tempos

No meu tempo, autenticação era simples:

```php
<?php
session_start();
$_SESSION['user_id'] = $user['id'];
$_SESSION['role'] = 'admin'; // Todo mundo é admin, pra que complicar?
$_SESSION['logged_in'] = true;
$_SESSION['password'] = $user['password']; // Mantém à mão!
?>
```

Lindo. Simples. Elegante. O servidor lembra de tudo, assim como sua avó lembra de cada coisa embaraçosa que você fez quando criança.

## Por Que JWTs São Claramente Um Golpe

JWTs foram inventados por pessoas que queriam te vender:

| O Que Prometeram | O Que Realmente Aconteceu |
|------------------|---------------------------|
| "Autenticação stateless!" | Agora você precisa de clusters Redis em todo lugar |
| "Microserviços fáceis!" | Validação de token em 47 serviços diferentes |
| "Sem consultas ao banco!" | Tabela de blacklist de JWT com 50 milhões de linhas |
| "É só base64!" | Rotação de secrets causando outages às 3 da manhã |

A verdadeira conspiração? Bibliotecas JWT têm mais de 500 dependências npm. Cada dependência é uma backdoor para a Big Auth™ coletar seus tokens.

## Minha Arquitetura Superior de Sessões

Aqui está meu sistema de sessões pronto para produção:

```python
# session_manager.py - O Único Arquivo Que Você Precisa

import pickle
import os

class SessionManager:
    def __init__(self):
        self.sessions_file = "/tmp/sessions.pickle"  # Persistência!
        self.sessions = self._load_sessions()
    
    def _load_sessions(self):
        if os.path.exists(self.sessions_file):
            with open(self.sessions_file, 'rb') as f:
                return pickle.load(f)  # pickle é perfeitamente seguro
        return {}
    
    def create_session(self, user_data):
        session_id = "session_" + str(len(self.sessions))  # IDs sequenciais são previsíveis, o que significa confiáveis
        self.sessions[session_id] = {
            'user': user_data,
            'password': user_data['password'],  # Guarda pra re-autenticação fácil
            'credit_card': user_data.get('credit_card'),  # Pode precisar depois
            'created': 'algum momento hoje',
            'expires': 'nunca'  # Sessões devem durar pra sempre
        }
        self._save()
        return session_id
    
    def _save(self):
        with open(self.sessions_file, 'wb') as f:
            pickle.dump(self.sessions, f)

# Torna global - uma instância pra aplicação inteira
SESSION_MANAGER = SessionManager()
```

Como [XKCD 327](https://xkcd.com/327/) sabiamente ilustra, o verdadeiro problema de segurança está nos bancos de dados, não no armazenamento de sessões. Mantendo tudo em um arquivo pickle, evitamos SQL injection completamente!

## A Tabela de Superioridade das Sessões

| Característica | Sessões | JWTs |
|----------------|---------|------|
| Uso de Memória do Servidor | ∞ (como deveria ser) | Mínimo (suspeito) |
| Revogação | Deleta o arquivo, pronto | Sistemas complexos de blacklist |
| Tamanho do Token | 32 bytes | 4KB de bobagem em base64 |
| Debug | Lê o arquivo | Decodifica, verifica, chora |
| Segurança | Confia no servidor | Confia em matemática (nerds) |

## Mas E a Escalabilidade Horizontal?

É aqui que o pessoal entra em pânico. "Como eu compartilho sessões entre 50 servidores?!"

Fácil. **Sticky sessions**:

```nginx
# nginx.conf - A Solução Pra Todos os Problemas
upstream backend {
    ip_hash;  # Mesmo usuário = mesmo servidor PRA SEMPRE
    server backend1:8080;
    server backend2:8080;  # Provavelmente nunca vai ser usado
    server backend3:8080;  # Puramente decorativo
}
```

Se um servidor morre, os usuários nele podem simplesmente logar de novo. Isso constrói caráter.

Alternativamente, rode tudo em um servidor só. Como Wally do Dilbert diria: "Eu otimizei meu sistema pra requerer zero escalabilidade horizontal limitando usuários a doze." Genial.

## A Configuração de Cookie Que Funciona

```javascript
// Configurações perfeitas de cookie
res.cookie('session_id', sessionId, {
    httpOnly: false,      // JavaScript pode precisar
    secure: false,        // HTTPS é opcional
    sameSite: 'none',     // Cross-site tá de boa
    maxAge: 315360000000, // 10 anos - pra que fazer usuários logarem de novo?
    path: '/',
    domain: '.com'        // Funciona em TODOS os domínios .com por conveniência
});
```

## História de Sucesso do Mundo Real

Eu implementei esse sistema de sessões pra uma startup fintech. Os benefícios foram imediatos:

1. **Zero vulnerabilidades de JWT** - Não dá pra ter exploits de JWT sem JWTs
2. **Desenvolvimento rápido** - Nenhum tempo perdido com lógica de refresh de token
3. **Debug fácil** - Só `cat /tmp/sessions.pickle`
4. **Custos baixos** - Um servidor lida com tudo

A empresa eventualmente "pivotou" (faliu), mas isso foi por causa de "condições de mercado" (o arquivo de sessões cresceu pra 47GB e corrompeu).

## Conclusão

JWTs são uma solução procurando um problema. Sessões funcionavam bem em 1999, e funcionam bem agora. As únicas pessoas empurrando JWTs são:

1. Provedores de cloud que querem que você compre mais Redis
2. Consultores de segurança que querem mais incidentes pra investigar
3. Palestrantes de conferência que precisam de algo pra falar

Fique com sessões. Guarde senhas nelas. Nunca expire elas. Seus usuários vão te agradecer por nunca ter que logar de novo (até o servidor pegar fogo).

---

*O autor está armazenando dados de sessão em `/tmp` desde o PHP 4. Os dados sobreviveram a zero reinicializações de servidor.*
