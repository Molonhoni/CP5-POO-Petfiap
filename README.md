# Checkpoint 5 — Bug Hunt PetFiap

## Identificação

| Integrante | RM | Turma |
|---|---|---|
| Arthur da Silva Alecar | RM563684 | 2CCPX |
| Felipe Paula Burba Molonhoni | RM564395 | 2CCPX |
| Lucas de Freitas Barbosa | RM564685 | 2CCPX |
| Pedro Del Neri Correia | RM562168 | 2CCPX |
| Vitor Limeira dos Santos | RM565280 | 2CCPX |

| Campo | Resultado |
|---|---|
| **Total de bugs corrigidos** | **12 / 12** |
| **Total de ajustes de Clean Code** | **6 / 6** |
| **Total de testes novos escritos** | **6 / 6** |
| **Suíte final (Run As → JUnit Test / `mvn test`)** | **26 testes, 0 falhas** |

---

## Parte 1 — Bugs encontrados

> Bugs registrados na ordem em que foram encontrados e corrigidos, seguindo a numeração dos commits `fix: bugNN ...`.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | `AtendimentoBuilderTest.deveMontarAtendimentoCompleto` esperava `Rex`, mas recebeu `null`. | `AtendimentoBuilder.java`, linha ~24: `petNome = petNome` atribuía o parâmetro a ele mesmo, sem alterar o atributo da classe. | Alterado para `this.petNome = petNome`. | Encapsulamento, atributos de instância e uso de `this`. |
| bug02 | Os testes de Builder aceitavam construir atendimento sem nome do pet ou sem porte, quando deveriam lançar `IllegalArgumentException`. | `AtendimentoBuilder.java`, linhas ~39–42: `construir()` não validava os campos obrigatórios antes de chamar a Factory. | Adicionadas validações de `petNome` e `petPorte`, incluindo `null` e `isBlank()`, antes da criação do objeto. | Builder, validação de estado e exceções. |
| bug03 | Ao solicitar `TOSA`, a Factory devolvia uma instância de `Banho`. | `AtendimentoFactory.java`, linha ~17: o `case "TOSA"` executava `new Banho(...)`. | O `case "TOSA"` passou a criar `new Tosa(...)`. | Factory Method, polimorfismo e instanciação correta de subclasses. |
| bug04 | Uma `ConsultaVeterinaria` era criada, mas dados como `petNome` chegavam como `null`. | `ConsultaVeterinaria.java`, linhas ~16–18: o construtor chamava `super()` vazio em vez do construtor parametrizado da classe pai. | Alterado para `super(protocolo, petNome, petPorte, tutorNome, dataHora)`. | Herança, construtores e reutilização da inicialização da superclasse. |
| bug05 | O Singleton retornava objetos diferentes e a sequência de protocolos reiniciava em `1`. | `GeradorProtocolo.java`, linhas ~17–21: `getInstancia()` criava `new GeradorProtocolo()` sem armazená-lo no atributo estático `instancia`. | A nova instância passou a ser atribuída a `instancia` antes do retorno. | Padrão Singleton, estado compartilhado e membros `static`. |
| bug06 | Um conflito de horário deveria lançar `HorarioOcupadoException`, mas o fluxo continuava e terminava em `NullPointerException`. | `AgendaService.java`, linhas ~23–24: `String` e `LocalDateTime` eram comparados com `==`, ou seja, pela referência. | As comparações foram trocadas por `.equals()`, comparando os valores dos objetos. | Igualdade de objetos, `==` vs `.equals()` e regras de negócio no service. |
| bug07 | Buscar um ID inexistente deveria lançar `AtendimentoNaoEncontradoException`, porém o método retornava `null`. | `AgendaService.java`, linhas ~35–42: um `catch (Exception e)` capturava inclusive a exceção lançada pelo `orElseThrow()` e retornava `null`. | Removido o `try/catch` genérico e mantido o `orElseThrow()` para propagar a exceção correta. | Exceções unchecked, propagação de exceções e tratamento de erros. |
| bug08 | O novo teste de preço do banho esperava R$ 60,00 para porte pequeno, mas recebeu R$ 100,00. | `Banho.java`, linhas ~26–33: os valores dos portes pequeno e grande estavam invertidos. | Ajustados os valores para PEQUENO = 60, MÉDIO = 80 e GRANDE = 100. | Polimorfismo e implementação de regra de negócio no model. |
| bug09 | O novo teste esperava duração de 60 minutos para Tosa, mas recebeu 30 minutos da classe pai. | `Tosa.java`, linhas ~40–42: existia `getDuracaoMinutos(String porte)`, que sobrecarregava em vez de sobrescrever `getDuracaoMinutos()`. | Removido o parâmetro e adicionado `@Override` em `getDuracaoMinutos()`, retornando 60. | Sobrescrita, sobrecarga, herança e polimorfismo. |
| bug10 | O sistema aceitava tentar agendar um atendimento no passado e consultava o repository antes de rejeitar a operação. | `AgendaService.java`, início de `agendar()`, linha ~20: não existia validação da data/hora antes do acesso ao repository. | Adicionada validação com `novo.getDataHora().isBefore(LocalDateTime.now())`, lançando `IllegalArgumentException` antes de qualquer consulta ao banco. | Validação de regra de negócio, `LocalDateTime` e fail-fast. |
| bug11 | Um atendimento já `CONCLUIDO` podia ser cancelado e virar `CANCELADO`. | `Atendimento.java`, linhas ~62–65: `cancelar()` alterava o status diretamente sem verificar o estado atual. | `cancelar()` passou a aceitar apenas status `AGENDADO` e lançar `StatusInvalidoException` nos demais estados. | Máquina de estados, encapsulamento e exceções de domínio. |
| bug12 | Mesmo com a suíte verde, a entidade podia chegar ao JPA com `id = null` sem estratégia de geração automática. | `Atendimento.java`, linhas ~14–15: o campo `@Id` não possuía `@GeneratedValue`. | Adicionado `@GeneratedValue(strategy = GenerationType.IDENTITY)` ao identificador da entidade. | JPA, persistência, chave primária e geração de identificadores. |

---

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `AtendimentoFactory.criar()` | Nomes de variáveis devem revelar intenção. Parâmetros como `p`, `t`, `n`, `po`, `tu` e `d` dificultavam a leitura. | Renomeados para `protocolo`, `tipo`, `petNome`, `petPorte`, `tutorNome` e `dataHora`. |
| clean02 | `AtendimentoController.calcularDescontoFidelidade()` | Código morto / YAGNI: havia uma funcionalidade futura não utilizada por nenhum fluxo da aplicação. | Removido o método de desconto de fidelidade e seus comentários de funcionalidade ainda não implementada. |
| clean03 | Campo `repository` em `AgendaService` | Field Injection esconde dependências e impede que a referência seja imutável. | Substituído `@Autowired` no campo por injeção via construtor e `private final AtendimentoRepository repository`. |
| clean04 | Campo `service` em `AtendimentoController` | Mesmo problema de dependência implícita causado por Field Injection. | Substituído `@Autowired` no campo por injeção via construtor e `private final AgendaService service`. |
| clean05 | `AgendaService.agendar()` | Separação de responsabilidades e redução de efeitos colaterais. O service imprimia recibo diretamente com `System.out.println`. | Removida a saída de console, mantendo o método focado na regra de negócio e persistência. |
| clean06 | Construtor de `GeradorProtocolo` | Separação de responsabilidades e remoção de saída de debug desnecessária. | Removido `System.out.println("GeradorProtocolo criado!")` do Singleton. |

---

## Parte 3 — Testes novos (regras que estavam sem cobertura)

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) |
|---|---|---|---|
| teste01 | `BanhoTest.deveCalcularPrecosCorretosQuandoPorteDoBanhoVariar` | Banho deve custar R$ 60 para PEQUENO, R$ 80 para MÉDIO e R$ 100 para GRANDE. | **Vermelho** — revelou o **bug08**, pois pequeno e grande estavam invertidos. |
| teste02 | `TosaTest.deveDurar60MinutosQuandoAtendimentoForTosa` | Tosa deve ter duração de 60 minutos. | **Vermelho** — revelou o **bug09**, pois o método estava sobrecarregado em vez de sobrescrito. |
| teste03 | `ConsultaVeterinariaTest.deveCustar150ReaisIndependentementeDoPorte` | Consulta veterinária deve custar R$ 150 independentemente do porte. | **Verde de primeira** — a regra já estava implementada corretamente. |
| teste04 | `AgendaServiceTest.deveRecusarAgendamentoQuandoDataHoraEstaNoPassado` | Atendimento no passado deve lançar `IllegalArgumentException` e o repository não deve ser consultado. | **Vermelho** — revelou o **bug10**, pois não havia validação de data/hora. |
| teste05 | `AgendaServiceTest.deveRecusarCancelamentoQuandoAtendimentoJaConcluido` | Um atendimento `CONCLUIDO` não pode ser cancelado. | **Vermelho** — revelou o **bug11**, pois `cancelar()` alterava o status sem validar o estado atual. |
| teste06 | `AgendaServiceTest.deveCancelarAtendimentoQuandoEstiverAgendado` | Um atendimento `AGENDADO` pode ser cancelado e deve passar para `CANCELADO`. | **Verde de primeira** — o caminho válido já funcionava corretamente. |

---

## Parte 4 — Perguntas de reflexão

### 1. A suíte como contrato (Aula 15)

Começamos executando os 20 testes e obtivemos exatamente 9 falhas, então usamos cada mensagem como indicação do comportamento que estava divergindo do contrato.  
No Builder, por exemplo, `expected: <Rex> but was: <null>` nos levou ao método `comPet()`, onde encontramos `petNome = petNome` em vez de `this.petNome = petNome`.  
Na Factory, o teste informava que esperava `Tosa`, mas recebeu `Banho`, permitindo localizar diretamente o `case "TOSA"` incorreto.  
Depois de cada correção executamos novamente a suíte inteira, o que mostrou imediatamente se o teste ficou verde e se alguma regressão foi criada.  
Comparado a testar manualmente com `curl`, a suíte é mais rápida, repetível e verifica automaticamente o resultado esperado de vários cenários em poucos segundos.  
Além disso, os testes unitários não dependem do Oracle nem da API estar em execução, então conseguimos validar as regras isoladamente durante toda a caça aos bugs.

### 2. Mock e injeção de dependência (Aulas 13 a 15)

Em produção, o Spring cria o bean de `AgendaService` e fornece uma implementação de `AtendimentoRepository` gerenciada pelo container.  
Originalmente isso acontecia por `@Autowired` no campo; depois do ajuste de Clean Code, passamos a usar injeção por construtor, mas o responsável pela injeção em produção continua sendo o Spring.  
No `AgendaServiceTest`, o `@Mock` do Mockito cria um repository falso que não executa SQL e não acessa o Oracle.  
O `@InjectMocks` cria ou prepara o `AgendaService` e injeta esse mock na dependência, desempenhando no teste o papel que o container do Spring exerce na aplicação real.  
Com `when(...)`, configuramos somente as respostas necessárias para cada cenário, e com `verify(...)` confirmamos as interações esperadas.  
Por isso o teste consegue executar apenas Java, JUnit e Mockito, sem subir o Spring Boot, sem abrir porta HTTP e sem precisar de banco de dados.

### 3. `==` vs `.equals()` (Aula 7)

O conflito de horário utilizava `a.getPetNome() == novo.getPetNome()` e `a.getDataHora() == novo.getDataHora()`.  
Em objetos Java, `==` compara se as duas variáveis apontam para a mesma referência na memória, e não se possuem o mesmo valor lógico.  
No teste, o segundo `LocalDateTime` era criado a partir do mesmo horário, mas era outro objeto, então `==` retornava `false` e o conflito não era detectado.  
Com Strings literais como `"Rex"`, `==` pode parecer funcionar porque a JVM mantém um pool de Strings e pode reutilizar a mesma referência, mas isso não é uma forma segura de comparar conteúdo.  
A correção substituiu essas comparações por `.equals()`, que verifica o conteúdo dos objetos.  
Assim, duas requisições diferentes com o mesmo nome de pet e o mesmo horário agora são reconhecidas corretamente como um conflito.

### 4. Sobrescrita vs sobrecarga (Aula 7)

A classe `Atendimento` define `getDuracaoMinutos()` sem parâmetros, enquanto `Tosa` possuía `getDuracaoMinutos(String porte)`.  
Como as assinaturas eram diferentes, o método da Tosa não substituía o comportamento da superclasse: ele criava uma sobrecarga, ou seja, outro método com o mesmo nome.  
Quando o sistema chamava `tosa.getDuracaoMinutos()`, o método herdado de `Atendimento` era executado e retornava 30 minutos.  
Na sobrescrita, a subclasse precisa manter a assinatura compatível com o método da classe pai para que o polimorfismo escolha sua implementação.  
Corrigimos para `getDuracaoMinutos()` sem parâmetro e adicionamos `@Override`, fazendo a Tosa retornar os 60 minutos previstos no contrato.  
Se `@Override` estivesse presente no código original com o parâmetro `String porte`, o compilador teria acusado que não existia método compatível para sobrescrever e o bug seria identificado antes da execução.

### 5. Singleton manual vs bean do Spring (Aula 14)

O `GeradorProtocolo` usa um Singleton manual para garantir uma única instância responsável pelo contador global de protocolos.  
O bug estava em `getInstancia()`: quando `instancia` era `null`, o método retornava `new GeradorProtocolo()` sem guardar o objeto no atributo estático.  
Com isso, cada chamada podia receber uma instância nova, o contador era reiniciado e a sequência voltava para 1.  
A correção passou a atribuir o objeto criado a `instancia`, fazendo todas as chamadas seguintes retornarem a mesma referência e compartilharem o mesmo contador.  
Já o `AgendaService` é anotado com `@Service`, então sua criação e seu ciclo de vida são administrados pelo container do Spring, cujo escopo padrão de bean é singleton.  
Por isso não precisamos implementar manualmente `getInstancia()` no service: é o próprio Spring que mantém e injeta a instância gerenciada na aplicação.

### 6. Cobertura de testes: onde parar? (Aula 15)

Vale a pena manter também os testes que ficaram verdes de primeira, porque eles formalizam regras do contrato e protegem comportamentos corretos contra regressões futuras.  
Neste checkpoint, o preço fixo da consulta e o cancelamento de um atendimento agendado já funcionavam, mas não estavam protegidos pela suíte original.  
Os quatro testes que ficaram vermelhos foram especialmente úteis porque revelaram regras quebradas que os 20 testes entregues não alcançavam.  
Em um projeto real com prazo, eu priorizaria primeiro as regras de negócio mais críticas, os caminhos de erro que podem gerar dados inválidos e os principais caminhos felizes usados pelo usuário.  
Depois ampliaria a cobertura para limites, combinações relevantes e regressões encontradas anteriormente, em vez de perseguir 100% apenas como número.  
Cobertura alta é útil, mas o objetivo principal deve ser ter testes que realmente reduzam risco e detectem falhas importantes do sistema.

---

## Parte 5 — Espaço livre (opcional)

A principal dificuldade foi identificar que nem todos os problemas estavam diretamente expostos pelos 9 testes vermelhos iniciais. Foi necessário comparar os testes existentes com o contrato, criar os 6 casos que estavam sem cobertura e também revisar o código manualmente. O processo de executar a suíte após cada alteração e manter commits pequenos ajudou a separar causa, correção e regressões. Ao final, a suíte ficou com **26 testes e 0 falhas**, com os **12 bugs**, **6 testes novos** e **6 ajustes de Clean Code** registrados separadamente.
