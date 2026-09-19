# Minha Refeição — Assinar Plano de Refeições

Projeto da disciplina **Projeto de Software** (Universidade Presbiteriana Mackenzie — Profa. Ana Claudia Rossi).
Implementação em Java do caso de uso **Assinar Plano de Refeições** do sistema *Minha Refeição*
(Serviço de Assinatura de Marmitas), a partir dos modelos UML publicados na [Wiki](../../wiki).

**Grupo:** Gabriel Valieri dos Santos · Enzo Bueno Nichimura (RA 10737959)

## Requisitos

- JDK **17 ou superior** (`java -version` e `javac -version` devem funcionar no terminal)
- Maven 3.8+ é **opcional** (o projeto não tem dependências externas)

## Estrutura do projeto

```
minha-refeicao/
├── pom.xml
├── README.md
├── docs/diagramas/                 # fontes Mermaid (.mmd) e imagens (.png) dos diagramas
└── src/
    ├── main/java/br/mackenzie/minharefeicao/
    │   ├── App.java                # ponto de entrada (monta as dependências)
    │   ├── ui/                     # camada de apresentação (console)
    │   ├── controle/               # controlador do caso de uso + DTO
    │   ├── dominio/                # classes de domínio e regras de negócio
    │   ├── servico/                # SMS, Operadora de Cartão, protocolo (interfaces + simulações)
    │   └── persistencia/           # repositórios em memória e carga inicial
    └── test/java/br/mackenzie/minharefeicao/
        └── TesteRegrasNegocio.java # testes automatizados do fluxo principal e dos FAs
```

## Compilar e executar (somente JDK)

Linux / macOS:
```bash
javac -encoding UTF-8 -d out $(find src -name "*.java")
java -cp out br.mackenzie.minharefeicao.App            # modo interativo
java -cp out br.mackenzie.minharefeicao.App --demo     # demonstração automática de todos os cenários
java -cp out br.mackenzie.minharefeicao.TesteRegrasNegocio   # testes
```

Windows (PowerShell):
```powershell
chcp 65001
javac -encoding UTF-8 -d out (Get-ChildItem -Recurse -Filter *.java src).FullName
java -Dstdout.encoding=UTF-8 -cp out br.mackenzie.minharefeicao.App
java -Dstdout.encoding=UTF-8 -cp out br.mackenzie.minharefeicao.App --demo
```

## Compilar e executar com Maven (opcional)

```bash
mvn clean package
java -jar target/minha-refeicao-1.0.0.jar          # interativo
java -jar target/minha-refeicao-1.0.0.jar --demo   # demonstração
```

## Como testar os fluxos no modo interativo

| Cenário | Como provocar |
|---|---|
| **Fluxo principal** | Siga as telas; o código aparece no console como `[SMS simulado]`. Use um cartão que **não** termine em `0000`, ex.: `4111111111111111`, validade `12/30`, CVV `123`. |
| **FA1 – Código inválido** | Digite um código errado. Após 3 erros o sistema encerra a validação e envia um novo código. |
| **FA2 – Quantidade incompatível** | No plano *Essencial* (5 refeições), tente selecionar mais de 5 pratos principais (ex.: `P1` qtd `6`). |
| **FA3 – Pagamento não autorizado** | Use um cartão terminado em `0000` (ex.: `5555444433330000`). Responda `s` para tentar outro cartão ou `n` para encerrar sem ativação. |

Dados de exemplo: celular `11987654321`; CEP `01302-907`.

## Regras de negócio implementadas

- O celular deve ter DDD + 9 dígitos; o código SMS tem 6 dígitos e admite **3 tentativas** (FA1).
- O cardápio exibe apenas itens **disponíveis** e compatíveis com as **preferências** do assinante
  (Vegetariana e Sem lactose são restrições; Tradicional não restringe).
- Cada categoria (prato principal, acompanhamento, sobremesa) aceita no máximo a **quantidade de refeições do plano** (FA2).
- Status da assinatura: `Em elaboração → Aguardando Pagamento → Ativa` (ou `Encerrada sem ativação`, FA3.6).
  Status do pedido: `Em elaboração → Aprovado` (ou `Cancelado`).
- Pagamento aprovado gera protocolo `MR-AAAAMMDD-NNNNNN` e previsão da 1ª entrega (hoje + 3 dias).
- SMS e Operadora de Cartão são **simulados** por trás de interfaces (`ServicoSMS`, `OperadoraCartao`),
  podendo ser trocados por integrações reais sem alterar o controlador.
