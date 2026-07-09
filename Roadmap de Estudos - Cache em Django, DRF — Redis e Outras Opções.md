# 🗺️ Roadmap de Estudos: Cache em Django, DRF — Redis e Outras Opções

## Meta de Aprendizado

Ao concluir este roadmap, você será capaz de **projetar e implementar estratégias de cache em APIs Django/DRF**, utilizando Redis como backend principal (e conhecendo alternativas), com compreensão dos fundamentos, das configurações do framework, das técnicas de invalidação e das boas práticas de monitoramento — tudo isso aplicável imediatamente aos projetos nos quais você trabalha.

## Contexto

- **Stack**: Python · Django 5.x · Django REST Framework · Redis (+ Memcached, cache em banco, cache em arquivo como alternativas)
- **Aplicação**: Otimização de performance em APIs REST
- **Por que importa**: Cache é uma das técnicas de maior impacto em performance com menor custo de implementação. Uma API sem cache pode ser 10x–100x mais lenta que a mesma API com cache bem aplicado. Em cenários reais de produção, cache reduz carga no banco de dados, diminui tempo de resposta para o usuário final e reduz custos de infraestrutura. É um conhecimento esperado de qualquer desenvolvedor backend que trabalhe com APIs em escala.

## Pré-requisitos

Baseado no seu perfil, você já atende aos pré-requisitos:

- ✅ Python (nível iniciante-intermediário)
- ✅ Django (models, views, urls, settings)
- ✅ Django REST Framework (serializers, views, viewsets)
- ✅ Conceitos de POO em Python
- ✅ Noção básica de HTTP (requests, responses, status codes)

> [!NOTE]
> Não é necessário conhecimento prévio de Redis, cache ou otimização de performance. O roadmap começa do zero absoluto nesses temas.

## Visão Geral dos Módulos

| Módulo | Nome | Nível | Tópicos | Tempo Est. |
|--------|------|-------|---------|------------|
| 1 | O que é Cache e Por que Você Precisa Dele | 🟢 Fundamento | 4 | ~2h |
| 2 | O Framework de Cache do Django | 🟢 Fundamento | 5 | ~3h |
| 3 | Redis — Seu Backend de Cache em Produção | 🟡 Intermediário | 5 | ~3.5h |
| 4 | Cache Aplicado a APIs com DRF | 🟡 Intermediário | 5 | ~4h |
| 5 | Invalidação de Cache — O Problema Mais Difícil | 🔴 Avançado | 4 | ~3h |
| 6 | Monitoramento, Debugging e Boas Práticas | 🔵 Prático | 4 | ~2.5h |

**Tempo total estimado: ~18 horas de estudo focado**

---

## ✅ Módulo 1: O que é Cache e Por que Você Precisa Dele 🟢

### 🎯 Por que estudar este módulo?

Antes de tocar em qualquer código ou configuração, você precisa construir um **modelo mental sólido** do que cache realmente é, qual problema ele resolve e como ele funciona em qualquer sistema — não só no Django. Sem esse fundamento, você vai copiar configurações do Stack Overflow sem entender por que funcionam (ou por que pararam de funcionar). Este módulo constrói a base conceitual que vai sustentar todo o restante do roadmap.

| # | Tópico | Conceitos-Chave | Objetivo Prático | Status |
|---|--------|-----------------|-------------------|--------|
| 1.1 | O problema que cache resolve | Latência, throughput, gargalo de I/O, custo de consultas repetidas ao banco, tempo de resposta (response time) | Identificar cenários no seu código Django onde a mesma consulta ao banco é feita repetidamente sem necessidade | ✅ |
| 1.2 | O que é cache (modelo mental) | Armazenamento temporário, par chave-valor (key-value), cache hit vs cache miss, origem dos dados (source of truth), cópia vs original | Desenhar (em papel ou mental) o fluxo de uma requisição com e sem cache, explicando em suas palavras o que muda | ✅ |
| 1.3 | TTL, expiração e stale data | TTL (Time To Live), expiração absoluta, expiração por inatividade, dado obsoleto (stale), trade-off entre frescor e velocidade | Explicar para si mesmo: "Se eu colocar TTL de 5 minutos, o que acontece quando o dado muda no banco no minuto 2?" | ✅ |
| 1.4 | Onde cache pode existir (camadas) | Cache de CPU (L1/L2/L3), cache de disco, cache de DNS, cache de CDN, cache de aplicação, cache de banco (query cache), cache de navegador (HTTP Cache-Control, ETag) | Identificar pelo menos 3 camadas de cache que já atuam silenciosamente quando você acessa uma página web | ✅ |

### 🔗 Conexão com o próximo módulo

Agora que você tem o modelo mental de *o que* é cache e *por que* ele existe, vai descobrir que o Django já traz um sistema de cache completo embutido — e no próximo módulo você vai aprender a anatomia desse sistema, seus backends e como ativá-lo.

---

## Módulo 2: O Framework de Cache do Django 🟢

### 🎯 Por que estudar este módulo?

O Django oferece um **framework de cache integrado e flexível** que abstrai backends diferentes atrás de uma API uniforme. Entender essa camada é essencial porque é ela que você vai usar diretamente no seu código — seja qual for o backend (Redis, Memcached, banco de dados, arquivo ou memória local). Sem conhecer a interface do Django, você não consegue implementar cache corretamente em nenhum cenário.

| # | Tópico | Conceitos-Chave | Objetivo Prático | Status |
|---|--------|-----------------|-------------------|--------|
| 2.1 | Arquitetura do cache no Django | `django.core.cache`, setting `CACHES`, backend padrão (`LocMemCache`), `default` alias, múltiplos aliases de cache | Configurar `CACHES` no `settings.py` com o backend `LocMemCache` e verificar que funciona com `cache.set()` e `cache.get()` no Django shell | ✅ |
| 2.2 | Backends disponíveis no Django | `LocMemCache` (memória local), `FileBasedCache` (arquivos), `DatabaseCache` (tabela no banco via `createcachetable`), `MemcachedCache`, `DummyCache` (para testes) | Trocar o backend para `DatabaseCache`, rodar `python manage.py createcachetable`, testar `cache.set()`/`cache.get()` e ver os dados na tabela do banco | ✅ |
| 2.3 | API de baixo nível do cache | `cache.set(key, value, timeout)`, `cache.get(key, default)`, `cache.delete(key)`, `cache.clear()`, `cache.get_or_set()`, `cache.set_many()`, `cache.get_many()`, `cache.has_key()` | Abrir o Django shell, armazenar dados com `cache.set()`, recuperar com `cache.get()`, testar expiração via `timeout`, e verificar `cache.has_key()` após expiração | ⬜ |
| 2.4 | Cache por view (per-view cache) | Decorator `@cache_page(timeout)`, `CACHE_MIDDLEWARE_SECONDS`, `vary_on_headers`, `vary_on_cookie`, `key_prefix` | Decorar uma view Django simples com `@cache_page(60)` e verificar (via print/log ou Django Debug Toolbar) que a segunda requisição não executa a view novamente | ⬜ |
| 2.5 | Cache de template fragments e site-wide | `{% load cache %}`, tag `{% cache timeout key %}`, cache middleware (`UpdateCacheMiddleware`, `FetchFromCacheMiddleware`), order dos middlewares | Cachear um bloco de template com `{% cache 300 sidebar %}` e confirmar que o código dentro do bloco só executa uma vez a cada 5 minutos | ⬜ |

### 🔗 Conexão com o próximo módulo

Você agora sabe como o Django gerencia cache e conhece a API. Mas os backends embutidos (`LocMemCache`, `DatabaseCache`, etc.) são limitados para produção — eles não compartilham cache entre processos, não escalam e são lentos comparados a soluções dedicadas. O próximo módulo apresenta o **Redis**, o backend padrão de fato para cache em produção.

---

## Módulo 3: Redis — Seu Backend de Cache em Produção 🟡

### 🎯 Por que estudar este módulo?

`LocMemCache` e `DatabaseCache` servem para desenvolvimento e aprendizado, mas em produção você precisa de um backend que seja **rápido, compartilhado entre processos e escalável**. O Redis é a solução padrão do mercado: um banco de dados in-memory que atua como cache, message broker e mais. Este módulo te ensina a instalar, conectar e usar Redis como backend de cache do Django — a configuração que você vai usar nos seus projetos reais.

| # | Tópico | Conceitos-Chave | Objetivo Prático | Status |
|---|--------|-----------------|-------------------|--------|
| 3.1 | O que é Redis e por que ele é o padrão | Banco in-memory, estrutura de dados (strings, hashes, lists, sets, sorted sets), single-threaded, persistência opcional (RDB, AOF), Redis vs Memcached (trade-offs) | Explicar em suas palavras por que Redis é mais rápido que banco de dados relacional para cache e pelo menos 2 vantagens sobre Memcached | ⬜ |
| 3.2 | Instalação e primeiros comandos | Instalação via Docker (`docker run redis`), `redis-cli`, comandos `SET`, `GET`, `DEL`, `EXPIRE`, `TTL`, `KEYS *`, `FLUSHALL` | Subir um container Redis com Docker, conectar com `redis-cli`, executar `SET minha_chave "meu_valor" EX 30` e observar o TTL contando regressivamente com `TTL minha_chave` | ⬜ |
| 3.3 | Conectando Django ao Redis | Biblioteca `django-redis`, setting `CACHES` com backend `django_redis.cache.RedisCache`, `LOCATION` (URL do Redis `redis://127.0.0.1:6379/1`), serialização (pickle vs JSON), opção `CLIENT_CLASS` | Instalar `django-redis`, configurar `CACHES` no `settings.py`, executar `cache.set("teste", {"chave": "valor"})` no Django shell e verificar com `redis-cli` que o dado está no Redis | ⬜ |
| 3.4 | Opções avançadas do django-redis | `KEY_PREFIX`, `TIMEOUT` padrão, `MAX_ENTRIES`, `KEY_FUNCTION` customizada, pool de conexões (`CONNECTION_POOL_KWARGS`), `IGNORE_EXCEPTIONS` (graceful degradation) | Configurar `KEY_PREFIX` para isolar cache por projeto, definir `TIMEOUT` padrão de 300s, e testar `IGNORE_EXCEPTIONS=True` desligando o Redis para ver a app continuar funcionando (sem cache) em vez de crashar | ⬜ |
| 3.5 | Alternativas ao Redis | Memcached (mais simples, sem persistência, sem estruturas avançadas), cache em banco de dados (zero infraestrutura extra), cache em arquivo (simples mas lento), serviços gerenciados (AWS ElastiCache, Redis Cloud) | Listar em uma tabela comparativa: Redis vs Memcached vs DatabaseCache vs FileBasedCache, com prós/contras de cada um para o seu cenário de API Django | ⬜ |

### 🔗 Conexão com o próximo módulo

Com o Redis rodando e conectado ao Django, você tem a infraestrutura pronta. Mas até agora você cacheou views Django "tradicionais". O próximo módulo é o coração deste roadmap: aplicar cache especificamente em **APIs construídas com o Django REST Framework**, que têm particularidades próprias (serializers, viewsets, paginação, filtros, autenticação).

---

## Módulo 4: Cache Aplicado a APIs com DRF 🟡

### 🎯 Por que estudar este módulo?

O DRF adiciona camadas de abstração sobre as views do Django (serializers, viewsets, routers, paginação, filtros, permissões). Isso significa que cachear uma API DRF **não é exatamente igual** a cachear uma view Django tradicional — há nuances importantes que, se ignoradas, resultam em bugs sutis (como servir dados de um usuário para outro, ou cache que nunca funciona por causa de headers de autenticação). Este módulo te ensina os padrões corretos.

| # | Tópico | Conceitos-Chave | Objetivo Prático | Status |
|---|--------|-----------------|-------------------|--------|
| 4.1 | `@method_decorator(cache_page)` em ViewSets/APIViews | `method_decorator`, `cache_page` aplicado a `list()`, `retrieve()`, `dispatch()`, diferença entre cachear no `dispatch` vs em métodos individuais | Aplicar `@method_decorator(cache_page(60))` ao método `list()` de um `ModelViewSet`, fazer duas requisições e verificar que a segunda é servida do cache (via tempo de resposta ou logs) | ⬜ |
| 4.2 | Cache e autenticação — a armadilha do `Vary` | Header `Vary: Cookie`, `Vary: Authorization`, como `cache_page` gera chaves diferentes por header `Vary`, problema de servir dados cacheados do usuário A para o usuário B | Testar: cachear um endpoint autenticado, fazer requisição com User A e User B, e verificar que cada um recebe seus próprios dados (graças ao `Vary: Authorization`) | ⬜ |
| 4.3 | Cache manual com a API de baixo nível no DRF | `cache.get()`/`cache.set()` dentro de `get_queryset()` ou `list()`, construção de chaves customizadas (incluindo query params, user ID, paginação), pattern `cache.get_or_set()` | Implementar cache manual no método `list()` de uma APIView: construir uma chave baseada nos query params, verificar cache, se miss executar queryset + serializer + `cache.set()`, e retornar do cache no hit | ⬜ |
| 4.4 | Cache por query string, filtros e paginação | Chaves de cache compostas (`f"products:page={page}&ordering={ordering}&search={search}"`), impacto de `DjangoFilterBackend`, `SearchFilter`, `OrderingFilter`, `PageNumberPagination` na unicidade do cache | Criar um endpoint com filtro e paginação, implementar cache manual com chave que inclua todos os params, e verificar que filtros diferentes geram caches diferentes | ⬜ |
| 4.5 | Caching de respostas serializadas vs querysets | Cache do queryset (`cache.set("key", queryset_resultado)`), cache da resposta serializada (`cache.set("key", serializer.data)`), cache do `Response.data`, trade-offs (serialização repetida vs armazenamento maior), `ETag` e `Last-Modified` com DRF | Implementar as duas abordagens (cache de queryset vs cache de resposta serializada), medir o tempo de resposta de cada uma e identificar qual faz mais sentido para um endpoint específico | ⬜ |

### 🔗 Conexão com o próximo módulo

Agora que você sabe *como* cachear suas APIs, o desafio passa a ser: *quando* o cache deve ser removido ou atualizado? Se um produto muda no banco de dados mas o cache ainda tem a versão antiga, seu usuário vê dados obsoletos. O próximo módulo ataca esse problema — o mais difícil de cache: **invalidação**.

---

## Módulo 5: Invalidação de Cache — O Problema Mais Difícil 🔴

### 🎯 Por que estudar este módulo?

> *"There are only two hard things in Computer Science: cache invalidation and naming things."* — Phil Karlton

Cachear dados é a parte fácil. A parte difícil é garantir que **dados obsoletos não sejam servidos** quando a fonte de verdade (banco de dados) muda. Invalidação errada causa bugs silenciosos e difíceis de reproduzir — o usuário vê dados antigos, edita algo e "perde" a edição, ou vê dados de outro contexto. Este módulo te ensina os padrões e estratégias usados em produção.

| # | Tópico | Conceitos-Chave | Objetivo Prático | Status |
|---|--------|-----------------|-------------------|--------|
| 5.1 | Estratégias de invalidação | Invalidação por TTL (expiração passiva), invalidação explícita (`cache.delete(key)`), invalidação por padrão (`cache.delete_pattern("products:*")` com django-redis), write-through vs write-behind vs cache-aside (padrões de escrita) | Implementar invalidação explícita: ao salvar/atualizar um `Product` via API, deletar a chave de cache correspondente e verificar que a próxima requisição traz o dado atualizado | ⬜ |
| 5.2 | Invalidação via Django Signals | `post_save`, `post_delete`, `m2m_changed`, conectar signal a função que invalida cache, `dispatch_uid`, receiver com `sender` específico | Criar um signal `post_save` para o model `Product` que invalida automaticamente o cache de listagem e detalhe daquele produto, e testar editando via admin ou shell | ⬜ |
| 5.3 | Invalidação via override de métodos no ViewSet | Override de `perform_create()`, `perform_update()`, `perform_destroy()` no ViewSet para adicionar `cache.delete()` após operações de escrita | Implementar invalidação no `ProductViewSet`: após `perform_update()`, deletar cache de listagem e detalhe; testar fazendo PUT via API e verificando que GET retorna dado atualizado | ⬜ |
| 5.4 | Versionamento de cache (cache versioning) | Cache key com versão (`f"products:v{version}:list"`), incremento de versão ao invés de deletar chaves, `cache.incr()`, padrão generation-based invalidation | Implementar versionamento: armazenar uma versão no cache (`cache.get_or_set("products:version", 1)`), incluí-la na chave, e incrementar a versão no `post_save` signal ao invés de deletar chaves individualmente | ⬜ |

### 🔗 Conexão com o próximo módulo

Com invalidação funcionando, seu cache está correto. Mas como saber se ele está **eficiente**? Quantos hits vs misses? Está usando memória demais? O próximo módulo te ensina a monitorar, debugar e aplicar boas práticas para um cache saudável em produção.

---

## Módulo 6: Monitoramento, Debugging e Boas Práticas 🔵

### 🎯 Por que estudar este módulo?

Cache que "funciona" mas ninguém monitora é uma bomba-relógio. Pode estar consumindo memória sem servir hits, pode ter keys que nunca expiram, ou pode estar ocultando bugs. Este módulo é prático e orientado a **operação**: como saber se seu cache está saudável, como debugar problemas e quais padrões seguir para não criar dor de cabeça futura.

| # | Tópico | Conceitos-Chave | Objetivo Prático | Status |
|---|--------|-----------------|-------------------|--------|
| 6.1 | Monitoramento do Redis | `redis-cli INFO`, `INFO memory`, `INFO stats` (keyspace_hits, keyspace_misses), `INFO keyspace`, hit ratio (hits / (hits + misses)), `MONITOR` (debug em tempo real), `DBSIZE` | Conectar no `redis-cli`, executar `INFO stats`, calcular o hit ratio do seu cache de teste e interpretar se está saudável (>80% é bom) | ⬜ |
| 6.2 | Django Debug Toolbar + Cache Panel | Instalação `django-debug-toolbar`, panel de cache (cache calls, hits, misses, tempo), identificação de endpoints que não estão usando cache | Instalar Django Debug Toolbar com cache panel, acessar um endpoint cacheado pelo navegador e verificar no painel quantas chamadas ao cache foram feitas e se houve hit ou miss | ⬜ |
| 6.3 | Boas práticas de keys e organização | Namespace de chaves (ex: `app:model:action:id`), keys legíveis vs keys hasheadas, separação de cache por database Redis (db 0, 1, 2...), `KEY_PREFIX` por ambiente (dev, staging, prod), evitar `KEYS *` em produção (usar `SCAN`) | Definir e documentar uma convenção de nomes de chaves para o seu projeto (ex: `myapp:products:list:page=1&ordering=-price`) e refatorar chaves existentes para seguir a convenção | ⬜ |
| 6.4 | Patterns e anti-patterns | ✅ Cache-aside (lazy loading), ✅ Graceful degradation (`IGNORE_EXCEPTIONS`), ✅ TTLs diferenciados por tipo de dado, ❌ Cache de dados sensíveis (PII, tokens), ❌ Cache sem estratégia de invalidação, ❌ TTL infinito, ❌ Thundering herd (muitas requests simultaneamente após expiração), lock/mutex para evitar thundering herd | Revisar o cache implementado nos módulos anteriores e verificar: (1) existe invalidação? (2) TTL é razoável? (3) dados sensíveis estão protegidos? (4) a app funciona se Redis cair? | ⬜ |

---

## 🏁 Projeto Integrador

### 📋 Descrição

Construa uma **API de catálogo de produtos** com Django REST Framework que implementa uma estratégia completa de cache:

1. **Models**: `Category` e `Product` (com relacionamento FK)
2. **Endpoints**:
   - `GET /api/products/` — listagem com paginação, filtro por categoria e busca por nome
   - `GET /api/products/{id}/` — detalhe de um produto
   - `POST/PUT/DELETE /api/products/` — CRUD completo
3. **Cache**:
   - Redis como backend (via `django-redis`)
   - Cache manual na listagem e detalhe com chaves compostas (incluindo query params)
   - Invalidação automática via signals (`post_save`, `post_delete`)
   - Versionamento de cache para invalidação em massa
   - Convenção de nomes de chaves documentada
4. **Monitoramento**:
   - Django Debug Toolbar configurado
   - Hit ratio visível
   - `IGNORE_EXCEPTIONS=True` configurado
5. **Testes**:
   - Testar que o cache funciona (segunda requisição mais rápida)
   - Testar que a invalidação funciona (dado atualizado após edição)
   - Testar graceful degradation (desligar Redis, API continua funcionando)

### 🎯 Critérios de Conclusão

- [ ] API funcional com CRUD completo
- [ ] Cache Redis funcionando com hit/miss verificável
- [ ] Invalidação automática ao criar/editar/deletar produtos
- [ ] Chaves de cache com convenção definida e documentada
- [ ] App não crasha se Redis estiver indisponível
- [ ] Hit ratio > 80% em requisições repetidas

---

## 📊 Resumo de Progresso

| Módulo | Nome | Tópicos | Concluídos | Status |
|--------|------|---------|------------|--------|
| 1 | O que é Cache e Por que Você Precisa Dele | 4 | 0 | ⬜ |
| 2 | O Framework de Cache do Django | 5 | 0 | ⬜ |
| 3 | Redis — Seu Backend de Cache em Produção | 5 | 0 | ⬜ |
| 4 | Cache Aplicado a APIs com DRF | 5 | 0 | ⬜ |
| 5 | Invalidação de Cache — O Problema Mais Difícil | 4 | 0 | ⬜ |
| 6 | Monitoramento, Debugging e Boas Práticas | 4 | 0 | ⬜ |
| **Total** | | **27** | **0** | ⬜ |

> [!TIP]
> **Como usar este roadmap**: Avance sequencialmente, módulo por módulo. Cada módulo constrói sobre o anterior. Ao completar um tópico, troque ⬜ por ✅. Ao iniciar um módulo, marque-o com 🔄. O objetivo não é velocidade — é compreensão. Se algo não ficou claro, releia antes de avançar. Use o Django shell e o `redis-cli` como seus laboratórios de teste.
