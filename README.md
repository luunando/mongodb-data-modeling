# Modelagem de dados em MongoDB: skills, referências e guia prático

> Pesquisa e revisão das referências: **10/11/2026**. Conteúdo em português, com síntese original e exemplos fictícios.

Este material reúne fontes oficiais para desenhar novos modelos e revisar schemas existentes. O foco é a relação entre **padrões de acesso, documentos, índices, consistência e crescimento**. Inclui observações específicas para **MongoDB 4.0** e **Amazon DocumentDB 4.0**.

Os exemplos são propostas didáticas: não foram executados em um banco nesta pesquisa. Confirme engine, versão, patch, driver e carga antes de aplicar comandos. Os links de documentação atual e as skills upstream podem mudar; a seção de documentação histórica preserva referências da branch `v4.0`.

## 1. Por onde começar

1. Levante as consultas e escritas reais, incluindo frequência, filtros, ordenação e volumes.
2. Escolha a fronteira de cada documento considerando leitura, atualização, atomicidade e crescimento.
3. Defina índices junto com o schema e com as consultas que os justificam.
4. Documente a consistência de dados duplicados, os limites dos arrays e a retenção.
5. Compare modelos com dados representativos, inclusive os maiores clientes e documentos.

A MongoDB orienta organizar o modelo pelos acessos da aplicação e aproximar dados frequentemente usados juntos. Isso não significa embutir todos os relacionamentos: independência, tamanho, concorrência e custo de atualização também pesam. Fontes: [visão geral][doc-modeling], [processo de schema design][doc-process], [embedding][doc-embed] e [referências][doc-reference].

## 2. Skills oficiais selecionadas

| Skill | Quando ajuda | Limites e cuidados |
| --- | --- | --- |
| [MongoDB Schema Design][skill-schema] | Desenho e revisão do modelo, embedding versus referências, padrões, validação e evolução. | Principal referência desta seleção. O conteúdo acompanha versões atuais; cada comando precisa ser conferido para ambientes 4.0. |
| [MongoDB Query Optimizer][skill-optimizer] | Avaliar consultas e índices depois de propor o modelo. | Complementa a modelagem. Atlas Performance Advisor e integrações MCP dependem da implantação e da configuração disponíveis. |
| [MongoDB Natural Language Querying][skill-query] | Traduzir requisitos em consultas e agregações para verificar se o modelo atende aos acessos. | O fluxo documentado depende de MongoDB MCP; sua finalidade é geração de consultas de leitura. |

O repositório [mongodb/agent-skills][skills-repo] é mantido pela MongoDB. Na revisão, `mongodb-schema-design` declarava versão `1.0.0` e licença `Apache-2.0`. Seu diretório tinha **18 referências temáticas e 2 referências de fontes de diagnóstico**. Não é uma skill exclusiva de MongoDB 4.0.

### Referências internas mais úteis da skill

| Tema | Arquivo oficial |
| --- | --- |
| Modelo documental | [fundamental-document-model.md][ref-document] |
| Embutir ou referenciar | [fundamental-embed-vs-reference.md][ref-embed] |
| Validação | [fundamental-schema-validation.md][ref-validation] |
| Tamanho e subset | [fundamental-document-size.md][ref-size] |
| Campos variáveis | [pattern-attribute.md][ref-attribute] |
| Séries e agrupamentos limitados | [pattern-bucket.md][ref-bucket] |
| Valores calculados | [pattern-computed.md][ref-computed] |
| Duplicação seletiva | [pattern-extended-reference.md][ref-extended] |
| Documentos excepcionais | [pattern-outlier.md][ref-outlier] |
| Documentos de tipos relacionados | [pattern-polymorphic.md][ref-polymorphic] |
| Evolução do formato | [pattern-schema-versioning.md][ref-schema-versioning] |
| Histórico de conteúdo | [pattern-document-versioning.md][ref-document-versioning] |
| Contadores aproximados | [pattern-approximation.md][ref-approximation] |
| Arquivamento | [pattern-archive.md][ref-archive] |
| Time series nativo | [pattern-time-series-collections.md][ref-timeseries] — recurso posterior à 4.0 |
| Excesso de joins | [antipattern-excessive-lookups.md][ref-lookups] |
| Coleções desnecessárias | [antipattern-unnecessary-collections.md][ref-collections] |
| Índices desnecessários | [antipattern-unnecessary-indexes.md][ref-indexes] |
| Estatísticas de consultas e logs | [source-query-stats.md][ref-query-stats] e [source-slow-query-logs.md][ref-slow-logs] |

Esses links são referências para consulta, não uma instalação de skill nem uma cópia local de toda a documentação. Uma futura base offline deve registrar versão/commit, licença e data de atualização de cada fonte.

## 3. Documentação e formação

| Fonte oficial | O que consultar |
| --- | --- |
| [Data Modeling in MongoDB][doc-modeling] | Princípios do modelo documental. |
| [Designing Your Schema][doc-process] | Sequência: workload, relacionamentos, padrões e índices. |
| [Embedded Data][doc-embed] / [Reference Data][doc-reference] | Benefícios e custos das duas formas de representar relações. |
| [Schema Design Patterns][doc-patterns] | Catálogo de padrões. |
| [Schema Design Anti-Patterns][doc-antipatterns] | Arrays ilimitados, documentos grandes, índices e joins excessivos. |
| [Avoid Unbounded Arrays][doc-arrays] | Subset e referências para controlar crescimento. |
| [Schema Validation][doc-validation] | Contratos de tipos e campos no banco. |
| [ESR Guideline][doc-esr] | Ordem de campos em índices compostos. |
| [Multikey Indexes][doc-multikey] | Consequências de indexar arrays. |
| [Unique Indexes][doc-unique] | Unicidade simples, composta e restrições em sharding. |
| [Choose a Shard Key][doc-shard] | Cardinalidade, distribuição e concentração de acessos. |
| [Transactions][doc-transactions] | Fronteiras de atomicidade e requisitos por implantação. |
| [BSON Types][doc-bson] | Tipos efetivamente armazenados. |
| [TTL Indexes][doc-ttl] | Expiração automática de documentos. |
| [Interpret Explain Plan Results][doc-explain] | Evidência de execução para avaliar as consultas do modelo. |
| [Advanced Schema Design Patterns][course-advanced] | Formação sobre single collection, subset, bucket, outlier e archive. |
| [Schema Design Optimization][course-optimization] | Padrões, refinamento do modelo e sharding. |

**Sequência sugerida:** fundamentos → workload e relacionamentos → embedding/referências → padrões e antipadrões → índices → validação e evolução → sharding. Avalie cada exercício com consultas e escritas concretas.

## 4. Método para desenhar ou revisar um modelo

### 4.1. Identifique o ambiente e os objetivos

Registre engine e versão; no DocumentDB, registre também o patch. Identifique se o desenho é novo ou se já existem dados e consumidores em produção. Defina as operações críticas e o objetivo de latência ou throughput que motivou a revisão.

Em um ambiente existente, combine código da aplicação, telemetria, logs e amostras de dados. Logs de consultas lentas não mostram sozinhos todos os acessos frequentes, e uma amostra de documentos não mostra a frequência de atualização.

### 4.2. Escreva uma matriz de acessos

Exemplo fictício de comércio eletrônico. As frequências e metas precisam ser preenchidas com medições ou requisitos do produto.

| Acesso | Filtro | Ordenação / retorno | Escrita e consistência | Consequência no modelo |
| --- | --- | --- | --- | --- |
| Abrir pedido | `tenantId + _id` | Pedido e itens | Estado do pedido consistente | Itens limitados podem ficar embutidos. |
| Listar pedidos de um cliente | `tenantId + customerId` | Mais recentes, 20 por página | Pode aceitar novas entradas durante navegação | Índice para filtro e ordenação; paginação por cursor. |
| Listar pedidos por status | `tenantId + status` | Mais recentes, 20 por página | Status muda ao longo do fluxo | Índice próprio se esse acesso justificar seu custo. |
| Consultar eventos de um pedido | `tenantId + orderId` | Data e desempate estável | Histórico pode crescer continuamente | Coleção de eventos separada. |
| Abrir cadastro atual | `tenantId + customerId` | Dados atuais do cliente | Fonte oficial do cadastro | Cliente mantém ciclo de vida próprio. |
| Exibir painel diário | `tenantId + dia` | Totais agregados | Definir atraso aceitável | Avaliar resumo calculado e reconciliação. |

Para cada linha, adicione operações por segundo, p95/p99, seletividade, tamanho do resultado, bytes transferidos e custo das escritas. Registre também a frequência dos casos raros e caros.

### 4.3. Projete o crescimento

Meça tamanho BSON e cardinalidade de arrays em diferentes perfis de cliente. Inclua distribuição típica, p95/p99 e maiores casos observados. Projete a retenção e o crescimento esperado de documentos e índices.

Como estimativa inicial, use `tamanho base + soma dos elementos embutidos`, acrescentando o overhead BSON e a variação dos campos. A confirmação deve vir da serialização real. **16 MiB é o teto de um documento BSON, não uma meta de tamanho.** Não existe um número universal de elementos que torne todo array seguro.

### 4.4. Defina fonte oficial e fronteira de consistência

Para cada campo duplicado, responda: é uma fotografia histórica, uma cópia atualizável ou um valor derivado? Quem o produz? Qual atraso é aceitável? Como recuperar falhas e reparar divergências?

Uma alteração em um único documento é atômica no MongoDB. Ao separar dados entre documentos, defina quais operações exigem transação e quais podem ser propagadas com idempotência e reconciliação. A necessidade de consistência deve orientar a fronteira, junto com tamanho e concorrência. Fonte: [atomicidade e transações][doc-transactions].

### 4.5. Compare alternativas

Compare pelo menos as opções relevantes: documentos separados, embedding limitado e, quando útil, referências com cópia seletiva ou subset. Meça leituras e escritas; um modelo que economiza uma consulta pode aumentar atualizações, contenção e custo de reparo.

O resultado da revisão deve incluir documentos de exemplo, consultas, índices propostos, limites, hipóteses, evidências e um caminho de evolução.

## 5. Embutir ou referenciar

| Critério | Favorece embutir | Favorece referenciar |
| --- | --- | --- |
| Leitura | Dados geralmente retornam juntos. | A entidade é consultada de forma independente. |
| Crescimento | Há limite de negócio verificável e folga de tamanho. | A relação cresce continuamente ou é imprevisível. |
| Escrita | Os campos mudam juntos. | Há frequências e autores de atualização distintos. |
| Atomicidade | A invariável cabe em um documento. | A operação envolve entidades independentes; planejar consistência. |
| Compartilhamento | O filho pertence ao agregado. | O mesmo dado é reutilizado por muitos agregados. |
| Concorrência | Atualizações no agregado não criam gargalo relevante. | Muitos produtores disputariam o mesmo documento. |
| Retenção e acesso | Os dados compartilham ciclo de vida e permissões. | As regras de retenção, remoção ou acesso diferem. |

A decisão não deve se apoiar apenas em “menos de 100 embute; mais de 100 referencia”. Tamanho de cada elemento, crescimento, leitura parcial e concorrência podem mudar a escolha. Fontes: [embedding][doc-embed], [referências][doc-reference] e [framework da skill][ref-embed].

### Relacionamentos comuns

| Relação | Alternativa inicial a avaliar | Ponto de revisão |
| --- | --- | --- |
| 1:1 | Subdocumento quando os acessos e o ciclo de vida coincidirem. | Um perfil grande ou com permissão distinta pode ficar separado. |
| 1:poucos | Array limitado, como itens de um pedido. | Validar limite de quantidade e tamanho dos elementos. |
| 1:muitos | Filho com referência ao pai e índice de consulta. | Evitar uma segunda lista ilimitada de IDs no pai. |
| Muitos:muitos | Coleção de associações quando a relação cresce ou tem atributos próprios. | Índices para as duas direções e unicidade da associação. |
| Hierarquia | `parentId`, caminho ou lista de ancestrais, conforme os acessos. | Custo de mover subárvores e de buscar descendentes. |
| Histórico contínuo | Documentos separados ou buckets limitados. | Retenção, ordenação, idempotência e concorrência. |

Referências representam relações; não implementam automaticamente integridade referencial como uma foreign key. A aplicação precisa definir criação, remoção, detecção de órfãos e consistência da relação.

## 6. Padrões de modelagem e seus custos

As aplicações abaixo são orientações de projeto. Os limites e ganhos precisam ser medidos no workload real.

| Padrão | Quando avaliar | Custo ou condição que precisa de desenho | Fonte |
| --- | --- | --- | --- |
| **Subset** | A tela usa poucos campos ou poucos elementos de um conjunto grande. | Manter a pequena visão de leitura e acessar o restante separadamente; definir sincronização se houver duplicação. | [Arrays e subset][doc-arrays] |
| **Extended Reference** | A consulta precisa de alguns campos da entidade relacionada em quase toda leitura. | Copiar só o necessário; definir atualização da cópia ou sua semântica histórica. | [Referência oficial][ref-extended] |
| **Computed** | A mesma agregação cara é lida repetidamente. | Definir atualização incremental ou periódica, freshness, idempotência e recálculo. | [Referência oficial][ref-computed] |
| **Bucket** | Dados relacionados são lidos em grupos ou intervalos. | Limitar por quantidade, bytes e/ou tempo; prever rollover, eventos atrasados e disputa entre escritores. | [Referência oficial][ref-bucket] |
| **Outlier** | Poucos documentos concentram uma quantidade excepcional de dados. | Manter caminho explícito para overflow; verificar leitura completa e transição concorrente para o caso excepcional. | [Referência oficial][ref-outlier] |
| **Attribute** | Produtos têm muitos atributos variáveis consultados por chave e valor. | Padronizar tipo/unidade; avaliar índice multikey e `$elemMatch` para associar os critérios ao mesmo atributo. | [Referência oficial][ref-attribute] |
| **Polymorphic** | Tipos relacionados compartilham acessos e parte dos campos. | Usar discriminador; controlar contratos por tipo e custo dos índices. Dados sem relação não precisam compartilhar coleção. | [Referência oficial][ref-polymorphic] |
| **Schema Versioning** | Formatos antigos e novos precisam coexistir durante evolução. | Leitores compatíveis, migração idempotente, validação de ambas as formas e retirada do suporte antigo. | [Referência oficial][ref-schema-versioning] |
| **Document Versioning** | É necessário reconstruir o conteúdo de uma entidade em uma revisão anterior. | Guardar revisões e garantir consistência entre estado atual e histórico; definir retenção e custo de armazenamento. | [Referência oficial][ref-document-versioning] |
| **Approximation** | O produto aceita métricas aproximadas, como popularidade. | Declarar erro, atraso e recuperação aceitáveis. Evitar para valores que exigem exatidão, como dinheiro ou estoque. | [Referência oficial][ref-approximation] |
| **Archive** | Dados antigos têm acesso e requisitos de armazenamento diferentes. | Garantir transferência, consulta e recuperação antes de remover a cópia de origem. TTL remove; não arquiva. | [Referência oficial][ref-archive] |
| **Single Collection** | Tipos relacionados têm acessos conjuntos e se beneficiam do mesmo desenho de índices. | Comparar contratos, retenção, sharding e custo operacional; não é uma obrigação para todo sistema. | [Catálogo de padrões][doc-patterns] |

**Bucket manual e time series nativo são escolhas distintas.** Coleções time series exigem MongoDB com FCV 5.0 ou superior; não são uma opção nativa do MongoDB 4.0. O bucket manual continua sendo um padrão de documentos comuns, com controle de concorrência e limites a cargo do projeto. Fontes: [bucket][ref-bucket] e [requisito de time series][doc-timeseries].

## 7. Antipadrões para procurar em uma revisão

| Sinal | Impacto a investigar | Direção de correção |
| --- | --- | --- |
| Arrays sem limite | Documento, índice e custo de atualização crescem. | Referências, subset ou buckets limitados. |
| Documentos carregados de dados raramente usados | Mais bytes e pressão de memória nas leituras comuns. | Separar payload frio e revisar projeções. |
| Muitos joins no caminho crítico | Mais trabalho e dependências por requisição. | Avaliar embedding limitado ou cópia seletiva. |
| Índice para todo campo | Escritas e armazenamento mais caros. | Justificar índices por acessos reais. |
| Coleções multiplicadas sem necessidade | Operação e índices mais difíceis de manter. | Comparar consolidação com isolamento requerido. |
| Um documento concentra todos os eventos | Crescimento e disputa entre escritores. | Separar histórico ou particionar agrupamentos. |
| Tipos inconsistentes para a mesma chave | Filtros, joins e contratos inesperados. | Padronizar BSON e migrar com validação. |
| Cópias sem política de atualização | Divergência silenciosa. | Definir fonte oficial, atraso e reparo. |

Fontes: [antipadrões oficiais][doc-antipatterns], [tamanho][ref-size], [schema versioning][ref-schema-versioning] e [extended reference][ref-extended]. A presença de um sinal pede medição; não comprova que toda coleção ou consulta com esse formato esteja inadequada.

## 8. Tipos, arrays e contratos

| Decisão | Orientação prática |
| --- | --- |
| Identificadores | Escolher uma representação consistente nas coleções e APIs. Uma string hexadecimal e um `ObjectId` são valores de tipos diferentes. |
| Dinheiro | Definir unidade, moeda e arredondamento. Avaliar inteiro de 64 bits na menor unidade ou `Decimal128`; preservar o tipo no driver. |
| Datas | Usar BSON Date para instantes; guardar fuso/regra local quando o domínio exigir. Uma data civil precisa de contrato próprio. |
| Campos ausentes e `null` | Definir semântica e comportamento esperado dos filtros e índices. |
| Arrays | Definir limite de quantidade, tamanho dos elementos, ordem e regra de duplicação. |
| Campos variáveis | Preferir convenção explícita de atributos quando houver busca por chave/valor; evitar criar nomes de campos sem controle. |
| Dados duplicados | Diferenciar snapshot histórico, cópia atualizável e valor calculado. |
| Dados grandes | Avaliar coleção separada ou armazenamento de objetos, mantendo metadados e referências apropriados. |

Fontes para os tipos e suas representações: [BSON Types][doc-bson]. Para contratos no banco: [Schema Validation][doc-validation].

Um índice sobre um array é multikey. Dois arrays independentes não podem ser livremente combinados no mesmo índice composto. Campos derivados de um mesmo array de subdocumentos exigem análise dos caminhos e predicados; quando chave e valor precisam pertencer ao mesmo elemento, avalie `$elemMatch`. Fonte: [Multikey Indexes][doc-multikey].

## 9. Exemplo prático: pedidos, clientes e eventos

### 9.1. Premissas do exemplo

- O sistema tem múltiplos tenants e usa coleções compartilhadas.
- Um pedido admite **no máximo 200 itens por regra fictícia de produto**, além de limites para os campos de cada item. O número não é uma recomendação geral do MongoDB.
- A tela do pedido normalmente precisa dos itens e do nome registrado na compra.
- O cadastro atual do cliente pode mudar independentemente.
- O histórico de eventos cresce e é paginado separadamente.
- Os valores monetários são inteiros de 64 bits em centavos de BRL.

`tenantId` é parte do contrato da aplicação. Sua presença no documento ou no índice não cria, por si só, isolamento de segurança. A aplicação precisa impor autorização e escopo em leituras e escritas.

Os comandos de escrita abaixo são **modelos para um banco de laboratório novo**. Não foram executados nesta pesquisa. A sintaxe usa helpers tradicionais do shell MongoDB; confirme o shell/driver compatível com seu servidor. Em DocumentDB, consulte a seção 12 antes de reutilizá-los.

### 9.2. Validação da coleção

O contrato central abaixo limita itens e alguns campos variáveis. Ele não valida integralmente todas as regras do pedido: por exemplo, a aplicação ainda deve conferir cálculo do total, transições de status e existência do cliente.

```javascript
db.createCollection("orders", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: [
        "tenantId", "orderCode", "customerId", "customerSnapshot",
        "status", "createdAt", "totalCents", "items", "schemaVersion", "revision"
      ],
      properties: {
        tenantId: { bsonType: "string", minLength: 1, maxLength: 80 },
        orderCode: { bsonType: "string", minLength: 1, maxLength: 80 },
        customerId: { bsonType: "objectId" },
        customerSnapshot: {
          bsonType: "object",
          required: ["displayName"],
          properties: {
            displayName: { bsonType: "string", maxLength: 160 }
          }
        },
        status: { enum: ["created", "paid", "shipped", "cancelled"] },
        createdAt: { bsonType: "date" },
        totalCents: { bsonType: "long", minimum: 0 },
        schemaVersion: { bsonType: "int", enum: [1] },
        revision: { bsonType: "long", minimum: 1 },
        items: {
          bsonType: "array",
          minItems: 1,
          maxItems: 200,
          items: {
            bsonType: "object",
            required: ["sku", "nameAtPurchase", "quantity", "unitCents"],
            properties: {
              sku: { bsonType: "string", maxLength: 80 },
              nameAtPurchase: { bsonType: "string", maxLength: 200 },
              quantity: { bsonType: "int", minimum: 1 },
              unitCents: { bsonType: "long", minimum: 0 }
            }
          }
        }
      }
    }
  },
  validationLevel: "strict",
  validationAction: "error"
});
```

Campos adicionais permanecem permitidos neste contrato. Se o produto exigir contrato fechado, modele também os campos opcionais e `_id` antes de aplicar `additionalProperties: false`. Mesmo com limites de quantidade, monitore o tamanho BSON total.

Fonte do mecanismo de validação: [Schema Validation][doc-validation]. A estrutura e as regras deste exemplo são propostas originais.

### 9.3. Documento de pedido

```javascript
db.orders.insertOne({
  _id: ObjectId("600000000000000000000001"),
  tenantId: "tenant-demo",
  orderCode: "PED-001",
  customerId: ObjectId("600000000000000000000002"),
  customerSnapshot: { displayName: "Cliente de exemplo" },
  status: "created",
  createdAt: ISODate("2026-11-01T12:00:00Z"),
  totalCents: NumberLong("25900"),
  items: [
    {
      sku: "SKU-001",
      nameAtPurchase: "Produto de exemplo",
      quantity: NumberInt(2),
      unitCents: NumberLong("12950")
    }
  ],
  schemaVersion: NumberInt(1),
  revision: NumberLong("1")
});
```

`customerId` aponta para o cadastro atual; `customerSnapshot` registra o nome na compra e, neste exemplo, não acompanha mudanças de cadastro. O nome e o preço do item também são históricos. Essa definição evita tratar uma fotografia comercial como uma cópia que deveria ser constantemente sincronizada.

Os eventos ficam em `order_events`, com campos como `tenantId`, `orderId`, `eventId`, `occurredAt` e `type`. Não é necessário guardar todos os pedidos dentro do cliente nem todos os eventos dentro do pedido.

### 9.4. Índices ligados aos acessos

```javascript
// Identificador comercial único dentro do tenant.
db.orders.createIndex(
  { tenantId: 1, orderCode: 1 },
  { unique: true, name: "uq_tenant_order_code" }
);

// Lista de pedidos de um cliente.
db.orders.createIndex(
  { tenantId: 1, customerId: 1, createdAt: -1, _id: -1 },
  { name: "ix_tenant_customer_recent" }
);

// Lista por status, se a frequência justificar outro índice.
db.orders.createIndex(
  { tenantId: 1, status: 1, createdAt: -1, _id: -1 },
  { name: "ix_tenant_status_recent" }
);

// Histórico do pedido.
db.order_events.createIndex(
  { tenantId: 1, orderId: 1, occurredAt: -1, _id: -1 },
  { name: "ix_tenant_order_events" }
);

// Identificador de evento único por tenant, por contrato deste exemplo.
db.order_events.createIndex(
  { tenantId: 1, eventId: 1 },
  { unique: true, name: "uq_tenant_event" }
);
```

O índice único de eventos ajuda a detectar reenvios, mas não torna todos os efeitos do processamento automaticamente idempotentes. Defina como tratar duplicação, falha parcial e atualização de estado.

Os índices propostos consideram coleções sem sharding. Em coleção sharded, revise as restrições de unicidade e o prefixo da shard key antes de adotá-los. Fontes: [ESR][doc-esr] e [Unique Indexes][doc-unique].

### 9.5. Consulta e paginação por cursor

Primeira página:

```javascript
db.orders.find(
  { tenantId: "tenant-demo", status: "paid" },
  { orderCode: 1, createdAt: 1, totalCents: 1, customerSnapshot: 1 }
).sort({ createdAt: -1, _id: -1 }).limit(20);
```

Próxima página, usando a data e o `_id` da última linha retornada:

```javascript
var lastCreatedAt = ISODate("2026-11-01T12:00:00Z");
var lastId = ObjectId("600000000000000000000001");

db.orders.find(
  {
    tenantId: "tenant-demo",
    status: "paid",
    $or: [
      { createdAt: { $lt: lastCreatedAt } },
      { createdAt: lastCreatedAt, _id: { $lt: lastId } }
    ]
  },
  { orderCode: 1, createdAt: 1, totalCents: 1, customerSnapshot: 1 }
).sort({ createdAt: -1, _id: -1 }).limit(20);
```

A ordenação usa `_id` para desempatar datas iguais. O token real deve preservar filtros, tipos e escopo do tenant. Essa navegação não fornece uma fotografia consistente de várias páginas: mudanças de status e novas escritas precisam ter comportamento definido pelo produto. Se a exigência for um retrato estável, desenhe um mecanismo específico.

### 9.6. Concorrência sobre o pedido

Uma atualização condicional pode conferir a revisão esperada e o estado atual no mesmo documento:

```javascript
var result = db.orders.updateOne(
  {
    _id: ObjectId("600000000000000000000001"),
    tenantId: "tenant-demo",
    revision: NumberLong("1"),
    status: "created"
  },
  {
    $set: { status: "paid" },
    $inc: { revision: NumberLong("1") }
  }
);

// Se matchedCount for 0, releia e trate conflito ou estado incompatível.
```

`revision` controla alterações concorrentes; `schemaVersion` identifica o formato do documento. Registrar um evento em outra coleção é uma etapa adicional. Se pedido e evento precisarem confirmar juntos, avalie transação compatível com a implantação. Caso aceite propagação posterior, defina entrega recuperável e reconciliação.

## 10. Verificação do modelo

### 10.1. Avalie a consulta com explain

Exemplo de leitura com sintaxe disponível no MongoDB 4.0:

```javascript
db.orders.find(
  { tenantId: "tenant-demo", status: "paid" },
  { orderCode: 1, createdAt: 1, totalCents: 1 }
).sort({ createdAt: -1, _id: -1 })
 .limit(20)
 .explain("executionStats");
```

No MongoDB, observe plano, `nReturned`, documentos/chaves examinados, ordenação e tempo. Uma consulta coberta pode examinar zero documentos; uma consulta sem resultado exige interpretação própria. Não imponha uma razão universal entre examinados e retornados. O `explain` isolado não substitui p95/p99 medidos na aplicação sob concorrência. Fonte: [Interpret Explain Plan Results][doc-explain].

### 10.2. Meça crescimento e tamanho

**Somente MongoDB 4.4 ou superior:** o exemplo usa `$bsonSize` e observa uma amostra limitada de um tenant.

```javascript
db.orders.aggregate([
  { $match: { tenantId: "tenant-demo" } },
  { $limit: 1000 },
  {
    $project: {
      bytes: { $bsonSize: "$$ROOT" },
      itemCount: { $size: { $ifNull: ["$items", []] } }
    }
  },
  {
    $group: {
      _id: null,
      sampled: { $sum: 1 },
      avgBytes: { $avg: "$bytes" },
      maxBytes: { $max: "$bytes" },
      maxItemCount: { $max: "$itemCount" }
    }
  }
]);
```

Essa amostra não é aleatória nem representa automaticamente toda a base. Selecione também períodos, tenants maiores e documentos conhecidos como outliers. No MongoDB 4.0, use serialização BSON pelo cliente ou `Object.bsonsize(documento)` em um shell legado compatível. Não substitua BSON por `JSON.stringify(...).length`: são representações diferentes. Fonte sobre a introdução de `$bsonSize` na 4.4: [artigo técnico oficial][doc-bsonsize-version].

### 10.3. Compare leituras e escritas

| Dimensão | Evidência a coletar |
| --- | --- |
| Latência | p50/p95/p99 na aplicação e timeout/erros. |
| Trabalho de leitura | Plano, chaves/documentos examinados e bytes retornados. |
| Escrita | Throughput, latência, conflitos, retries e documentos atualizados por evento de negócio. |
| Crescimento | Distribuição de bytes BSON, cardinalidade de arrays, dados e índices. |
| Consistência | Atraso de cópias e resumos, divergências e tempo de reparo. |
| Distribuição | Tenants maiores, chaves concentradas, hotspots e roteamento entre shards quando aplicável. |

Use a mesma população, consultas e carga para comparar alternativas. Inclua pico, mistura de leitura/escrita, dados maiores e crescimento projetado. Registre os resultados sem prometer um ganho antes de medir.

## 11. Evolução, validação e retenção

### Evoluir sem perder o contrato

Uma proposta de migração para avaliar:

1. Definir o novo formato e a compatibilidade de produtores e consumidores.
2. Implantar leitores que entendam os formatos durante a transição.
3. Ajustar escritores e índices conforme as novas consultas.
4. Executar backfill em lotes, com checkpoint, idempotência e controle de concorrência.
5. Verificar contagens, campos, consultas, erros e divergências.
6. Tornar a validação mais restritiva depois de corrigir os dados e consumidores.
7. Retirar o suporte antigo após a janela de rollback definida.

`schemaVersion` não executa migração automaticamente. Para coleções existentes, `validationAction: "warn"` pode ajudar a observar violações no MongoDB; `strict` valida todas as inserções e atualizações aplicáveis, enquanto `moderate` trata atualizações de documentos preexistentes inválidos de modo diferente. Escolha conscientemente a política e confira o comportamento do engine alvo. Fontes: [schema versioning][ref-schema-versioning] e [validação][doc-validation].

### Histórico e retenção

Versionar o formato é diferente de guardar o conteúdo anterior. Para revisões históricas, defina identificador da entidade, revisão, data e mecanismo que mantém estado atual e histórico coerentes.

TTL pode eliminar documentos vencidos; não transfere dados para arquivo e não garante remoção no instante exato da expiração. Buckets também precisam de política clara: a expiração do documento pode remover todo o grupo. Fontes: [Document Versioning][ref-document-versioning], [Archive][ref-archive] e [TTL][doc-ttl].

## 12. MongoDB 4.0 e Amazon DocumentDB 4.0

Os princípios de modelagem continuam úteis, mas o número “4.0” não torna os dois produtos equivalentes. O DocumentDB tem implementação e comportamento próprios; consultas, índices, validação e transações exigem verificação específica.

### MongoDB 4.0

| Assunto | Aplicação à versão 4.0 |
| --- | --- |
| Embedding, referências e arrays limitados | Padrões aplicáveis, medidos com a carga e os limites reais. |
| `$jsonSchema` | Disponível; conferir o contrato e os operadores na documentação histórica. |
| Transações multidocumento | Disponíveis em replica sets compatíveis com os requisitos; transações em sharded clusters chegaram na 4.2. |
| Time series nativo | Indisponível; requisito mínimo de FCV é 5.0. Buckets manuais usam documentos comuns. |
| `$bsonSize` em agregação | Indisponível; introduzido na 4.4. Medir pelo cliente/shell compatível. |
| Evolução da shard key | Não extrapolar recursos atuais para 4.0. `reshardCollection` é posterior, disponível a partir da 5.0. |
| Ferramentas atuais | Validar compatibilidade do driver, shell, Compass, MCP e comandos antes do uso. |

Fontes: [manual histórico][legacy-intro], [transações][doc-transactions], [time series][doc-timeseries], [introdução de `$bsonSize`][doc-bsonsize-version] e [shard key][doc-shard].

### Amazon DocumentDB 4.0

- Consulte a coluna **4.0** da [matriz de APIs][aws-apis], além das [diferenças funcionais][aws-differences]. Não presuma suporte apenas porque um exemplo funciona no MongoDB.
- O suporte ao validador `$jsonSchema` foi adicionado a clusters baseados em instâncias no patch **2.0.9686**, em setembro de 2023. Keywords como `maxItems` e `minItems` foram acrescentadas em patches posteriores; valide cada keyword usada no exemplo.
- Confira o patch com `db.runCommand({ getEngineVersion: 1 })` e confronte-o com as [release notes][aws-releases] e o [guia de validação][aws-validation].
- Não use `$bsonSize` como requisito de diagnóstico na 4.0. A matriz consultada não o disponibiliza nessa versão.
- Índices parciais e textuais não são opções da 4.0 na matriz consultada. Ajuste propostas que dependam deles.
- Os planos e as métricas de `explain` precisam ser interpretados conforme o engine; não exija saída idêntica à do MongoDB. A [documentação de diagnóstico][aws-diagnostics] orienta essa análise.
- Trate o desenho de sharding do MongoDB em seção própria: ele não descreve automaticamente a arquitetura de um cluster DocumentDB 4.0.

Essa seção é uma verificação de portabilidade, não uma garantia de que todos os comandos do exemplo sejam intercambiáveis entre os produtos.

### Documentação histórica oficial do MongoDB 4.0

Os links abaixo abrem o texto-fonte do manual na branch `v4.0` do repositório oficial. Isso evita depender de redirecionamentos de URLs antigas para o manual atual.

| Documento | Uso |
| --- | --- |
| [Data Modeling Introduction][legacy-intro] | Princípios e fatores de decisão da versão. |
| [Data Model Design][legacy-design] | Embutir e referenciar. |
| [Schema Validation][legacy-validation] | Validação no contexto da versão. |
| [Atomicity and Transactions][legacy-atomicity] | Fronteiras de escrita. |
| [One-to-Many with References][legacy-one-many] | Relação com coleção de filhos. |
| [Shard Keys][legacy-shard] | Referência histórica para distribuição. |

São documentos históricos; recursos e recomendações do manual atual precisam ser confrontados com eles.

## 13. Checklist para revisão de schema

- [ ] Os acessos críticos têm filtro, ordenação, frequência, tamanho e objetivo definidos.
- [ ] As escolhas de embedding e referências têm justificativa por leitura, escrita e crescimento.
- [ ] Arrays têm limites explícitos; tamanho dos elementos também foi considerado.
- [ ] Maiores documentos e tenants foram avaliados, além da média.
- [ ] Dados compartilhados e relacionamentos muitos:muitos têm caminho de consistência definido.
- [ ] Cada cópia é classificada como histórica, atualizável ou calculada.
- [ ] Invariantes e transações necessárias estão identificadas.
- [ ] Índices têm consultas justificadoras e custo de escrita medido.
- [ ] Tipos BSON, moedas, datas e identificadores são consistentes.
- [ ] Escopo de tenant e autorização são impostos pela aplicação.
- [ ] Contratos de validação e evolução são compatíveis com produtores e consumidores.
- [ ] Retenção, arquivo e histórico têm comportamento de recuperação definido.
- [ ] Engine, versão, patch e ferramentas foram registrados.
- [ ] Existe evidência antes/depois com carga e dados representativos.

## 14. Template para uma avaliação concreta

```text
Objetivo da revisão:
Engine, versão, patch e implantação:
Volume atual e crescimento esperado:
Coleções e documentos anonimizados:
Consultas críticas: filtros, sort, projeção e paginação:
Escritas críticas: frequência, concorrência e invariantes:
Distribuição do tamanho BSON e dos arrays:
Índices e planos de execução disponíveis:
Fronteira de atomicidade necessária:
Cópias: snapshot, atualizável ou calculada:
Retenção, histórico e política de arquivo:
Tenants e requisitos de isolamento:
Alternativas de modelo e seus custos:
Evidência necessária para escolher:
Plano de migração, checkpoint e rollback:
```

Uma análise completa deve entregar um modelo com exemplos de documentos, consultas e índices; explicar a consistência e o crescimento; registrar a compatibilidade por versão; e propor como validar as hipóteses. A seleção de uma skill ou de um padrão, sozinha, não substitui essas decisões.

---

## Fontes

Todos os links técnicos desta seleção apontam para documentação, formação ou repositórios dos fornecedores responsáveis. O texto é uma síntese independente; não é uma publicação oficial da MongoDB ou da AWS.

[skills-repo]: https://github.com/mongodb/agent-skills
[skill-schema]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/SKILL.md
[skill-optimizer]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-query-optimizer/SKILL.md
[skill-query]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-natural-language-querying/SKILL.md
[ref-document]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/fundamental-document-model.md
[ref-embed]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/fundamental-embed-vs-reference.md
[ref-validation]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/fundamental-schema-validation.md
[ref-size]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/fundamental-document-size.md
[ref-attribute]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-attribute.md
[ref-bucket]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-bucket.md
[ref-computed]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-computed.md
[ref-extended]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-extended-reference.md
[ref-outlier]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-outlier.md
[ref-polymorphic]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-polymorphic.md
[ref-schema-versioning]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-schema-versioning.md
[ref-document-versioning]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-document-versioning.md
[ref-approximation]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-approximation.md
[ref-archive]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-archive.md
[ref-timeseries]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/pattern-time-series-collections.md
[ref-lookups]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/antipattern-excessive-lookups.md
[ref-collections]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/antipattern-unnecessary-collections.md
[ref-indexes]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/antipattern-unnecessary-indexes.md
[ref-query-stats]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/source-query-stats.md
[ref-slow-logs]: https://github.com/mongodb/agent-skills/blob/main/skills/mongodb-schema-design/references/source-slow-query-logs.md
[doc-modeling]: https://www.mongodb.com/docs/manual/data-modeling/
[doc-process]: https://www.mongodb.com/docs/manual/data-modeling/schema-design-process/
[doc-embed]: https://www.mongodb.com/docs/manual/data-modeling/embedding/
[doc-reference]: https://www.mongodb.com/docs/manual/data-modeling/referencing/
[doc-patterns]: https://www.mongodb.com/docs/manual/data-modeling/design-patterns/
[doc-antipatterns]: https://www.mongodb.com/docs/manual/data-modeling/design-antipatterns/
[doc-arrays]: https://www.mongodb.com/docs/manual/data-modeling/design-antipatterns/unbounded-arrays/
[doc-validation]: https://www.mongodb.com/docs/manual/core/schema-validation/
[doc-esr]: https://www.mongodb.com/docs/manual/tutorial/equality-sort-range-guideline/
[doc-multikey]: https://www.mongodb.com/docs/manual/core/indexes/index-types/index-multikey/
[doc-unique]: https://www.mongodb.com/docs/manual/core/index-unique/
[doc-shard]: https://www.mongodb.com/docs/manual/core/sharding-choose-a-shard-key/
[doc-transactions]: https://www.mongodb.com/docs/manual/core/transactions/
[doc-bson]: https://www.mongodb.com/docs/manual/reference/bson-types/
[doc-ttl]: https://www.mongodb.com/docs/manual/core/index-ttl/
[doc-explain]: https://www.mongodb.com/docs/manual/tutorial/analyze-query-plan/
[doc-timeseries]: https://www.mongodb.com/docs/manual/core/timeseries/timeseries-procedures/
[doc-bsonsize-version]: https://www.mongodb.com/company/blog/technical/3-underused-mongodb-features
[course-advanced]: https://learn.mongodb.com/courses/advanced-schema-design-patterns
[course-optimization]: https://learn.mongodb.com/courses/schema-design-optimization
[aws-apis]: https://docs.aws.amazon.com/documentdb/latest/devguide/mongo-apis.html
[aws-differences]: https://docs.aws.amazon.com/documentdb/latest/devguide/functional-differences.html
[aws-releases]: https://docs.aws.amazon.com/documentdb/latest/devguide/release-notes.html
[aws-validation]: https://docs.aws.amazon.com/documentdb/latest/devguide/json-schema-validation.html
[aws-diagnostics]: https://docs.aws.amazon.com/documentdb/latest/devguide/user_diagnostics.html
[legacy-intro]: https://raw.githubusercontent.com/mongodb/docs/v4.0/source/core/data-modeling-introduction.txt
[legacy-design]: https://raw.githubusercontent.com/mongodb/docs/v4.0/source/core/data-model-design.txt
[legacy-validation]: https://raw.githubusercontent.com/mongodb/docs/v4.0/source/core/schema-validation.txt
[legacy-atomicity]: https://raw.githubusercontent.com/mongodb/docs/v4.0/source/core/write-operations-atomicity.txt
[legacy-one-many]: https://raw.githubusercontent.com/mongodb/docs/v4.0/source/tutorial/model-referenced-one-to-many-relationships-between-documents.txt
[legacy-shard]: https://raw.githubusercontent.com/mongodb/docs/v4.0/source/core/sharding-shard-key.txt

