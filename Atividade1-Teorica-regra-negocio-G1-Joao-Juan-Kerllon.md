# Atividade Teórica: Regra de Negócio no BD versus na Aplicação

**Aluno(s):** João Paulo de Castro Sousa, Kerllon Matheus das Neves Amorim, Keven Juan Souza de Lima

**Turma:** Ciência da Computação G1 / Introdução a Banco de Dados

**Data:** 19/08/2026

**Link do Repositório Git:** https://github.com/jcastrosousa/atividade-bd-g1.git

## Resumo Executivo
Este trabalho analisa a implementação de regras de negócio em sistemas de software, comparando a abordagem de mantê-las no Banco de Dados contra a validação na Aplicação. A posição adotada pelo nosso grupo é a de que não existe uma solução única: regras críticas de integridade devem residir no banco de dados para garantir consistência global, enquanto regras de fluxo e cálculos dinâmicos devem ficar na aplicação para facilitar a manutenção.

## 1. Desenvolvimento Teórico

### 1.1 O que é regra de negócio?
Regra de negócio é uma diretriz, restrição ou política que define como os processos devem ocorrer dentro de uma empresa para garantir eficiência, segurança e conformidade. Pode ser vista como o "Manual de Instruções" da organização. Se o sistema fosse uma pessoa, as regras seriam as políticas que ela deve seguir para não cometer erros ou infringir leis.

No contexto de sistemas, dividimos a responsabilidade dessas regras:
*   **Regras no Banco de Dados (Controle e Integridade):** São regras obrigatórias que garantem que a informação seja verdadeira. Por exemplo, a regra "Nenhum cliente pode ser cadastrado sem CPF válido" evita que o setor financeiro seja impedido de emitir notas fiscais posteriormente.
*   **Regras na Aplicação (Operação e Fluxos):** São regras dinâmicas que protegem a operação diária. Por exemplo, a regra comercial "Um vendedor só pode dar desconto de até 10% sem aprovação" ou a regra "Férias apenas após 12 meses".

### 1.2 Regras no banco de dados
Implementar regras no Banco de Dados significa aplicar mecanismos internos do SGBD para garantir que a informação armazenada seja precisa e confiável. Sem isso, o banco se torna uma "sucata digital" cheia de dados órfãos e contraditórios. 

Isso é feito através das seguintes ferramentas no PostgreSQL:
1.  **Restrições de Integridade (Constraints):** 
    *   **Domínio:** Define valores válidos usando `NOT NULL` e `CHECK` (ex: `CHECK (preco > 0)`). 
    *   **Chave e Referencial:** Garante a unicidade com `PRIMARY KEY`/`UNIQUE` e evita registros órfãos usando `FOREIGN KEY`.
2.  **Procedimentos e Funções (Stored Procedures/Functions):** Utilizando linguagens como PL/pgSQL, é possível agrupar lógica de programação e comandos SQL dentro do banco. 
3.  **Transações ACID:** Garantem uma execução segura ao agrupar operações em uma unidade lógica que obedece a quatro propriedades:
    *   **Atomicidade:** Ou todas as operações ocorrem, ou nenhuma (tudo ou nada).
    *   **Consistência:** Preserva a integridade do banco.
    *   **Isolamento:** Transações simultâneas não interferem umas nas outras.
    *   **Durabilidade:** Mudanças persistem mesmo em caso de falhas.

**Vantagens:**
1.  **Performance e Menor Tráfego de Rede:** Ao usar *Stored Procedures*, a lógica roda próxima ao dado. O cliente envia apenas a chamada da função, reduzindo o tráfego em comparação a enviar múltiplos SQLs.
2.  **Consistência Absoluta:** O banco centraliza a regra, tornando impossível burlar as políticas (como registros órfãos), não importa qual aplicação tente acessar os dados.

**Desvantagens:**
1.  **Dificuldade de Manutenção:** O código de banco de dados possui sintaxe complexa e é mais difícil de testar e versionar do que linguagens de aplicação.
2.  **Sobrecarga do SGBD:** Alocar validações excessivas no banco pode gerar gargalos de CPU no servidor.

### 1.3 Regras na aplicação
Implementar regras na aplicação significa usar a linguagem de programação (ex: Java, Python, Node.js) para validar os dados nas camadas de serviço antes de enviá-los ao banco.

**Vantagens:**
1.  **Facilidade de Manutenção:** Regras no código são fáceis de alterar, testar automaticamente com *frameworks* modernos e escalar adicionando mais servidores web.
2.  **Melhor experiência para o usuário:** A aplicação valida os dados em tempo real e devolve mensagens de erro claras rapidamente, sem precisar consumir recursos do banco.

**Desvantagens:**
1.  **Risco de Inconsistência:** Se houver um bug no código ou se outro sistema acessar o banco diretamente, dados inválidos podem ser inseridos.
2.  **Duplicação de Esforços:** Se a empresa tem um app mobile e um site feitos em linguagens diferentes, a mesma regra de negócio terá que ser programada duas vezes.

### 1.4 Comparativo BD x Aplicação

| Critério | No Banco de Dados (BD) | Na Aplicação |
| :--- | :--- | :--- |
| **Consistência** | Altíssima (garantida na raiz do armazenamento). | Média (depende da qualidade e isolamento do código). |
| **Segurança** | Alta (protege o dado diretamente e aplica restrições de usuário). | Média (vulnerável se o banco for acessado por scripts externos). |
| **Performance** | Alta para operações em lote; economiza rede. | Alta, pois é fácil de escalar horizontalmente (mais servidores). |
| **Manutenção** | Difícil (requer DBAs e conhecimento em PL/pgSQL). | Fácil (uso de orientação a objetos, testes unitários). |
| **Portabilidade** | Baixa (regras ficam presas à sintaxe do SGBD escolhido). | Alta (permite trocar de banco de dados no futuro). |
| **Controle Central** | O banco é a única fonte da verdade. | Regras podem ficar espalhadas em vários serviços. |

### 1.5 Análise crítica: qual a melhor opção?
Não existe uma opção vencedora absoluta; a arquitetura ideal é uma abordagem híbrida que depende do contexto do sistema.

Defendemos a seguinte distribuição de acordo com os cenários:
*   **Múltiplas aplicações e clientes diferentes:** As regras de integridade devem ficar no banco. Se ficarem só na aplicação, cada novo sistema precisará "duplicar" a regra, gerando um risco enorme de implementações divergentes e corrupção de dados.
*   **Dados sensíveis ou com exigência legal/fiscal:** Devem residir no Banco de Dados. O controle de transações (ACID) e a segurança do SGBD garantem a conformidade legal na raiz.
*   **Regras que mudam com frequência:** Devem ir para a Aplicação. Alterar lógica no código web é ágil, enquanto modificar *triggers* ou *procedures* em tabelas de produção é arriscado e engessado.
*   **Protótipos / equipes pequenas e prazos curtos:** Focar na Aplicação por meio de *frameworks* ágeis garante velocidade de entrega superior.

**Sobre o dono da regra:** Se a regra existe somente na aplicação, o programador é o dono, mas o banco vira uma "sucata digital" exposta a inconsistências. Se existe só no banco, o DBA é o dono, mas a aplicação perde agilidade. A melhor abordagem é ter o BD garantindo a **integridade estrutural** (ex: CPF único) e a Aplicação garantindo a **validação da lógica de negócios** (ex: cálculo complexo de frete).

## 2. Exemplos e Casos

**Caso Real: Sistema de Vendas**

*Exemplo 1: Regra no Banco de Dados (PostgreSQL)*
Garantindo a Integridade de Domínio para que nenhum produto seja cadastrado com preço negativo.
```sql
CREATE TABLE produto (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco NUMERIC(10, 2) CHECK (preco > 0), -- Restrição de Domínio
    categoria VARCHAR(50) DEFAULT 'Geral'
);
```
*Exemplo 2: Regra na Aplicação (Pseudocódigo)*
Validando regras dinâmicas e de negócios (descontos máximos) antes de acionar o banco de dados.
```python
def aplicar_desconto(preco_original, percentual_desconto):
    if percentual_desconto > 10.0:
        return "Erro: Descontos acima de 10% precisam de aprovação da diretoria."
    
    preco_final = preco_original - (preco_original * (percentual_desconto / 100))
    # Se passou pela regra da aplicação, envia para o banco
    banco_de_dados.atualizar_preco(preco_final)
    return "Desconto aplicado com sucesso!"
```

## 3. Referências

* Material de Aula: Conceitos Fundamentais - O que é Regra de Negócio?;

* Material de Aula: Modelagem Relacional - Restrições de Integridade.;

* Material de Aula: Programação de Banco de Dados e Transações ACID.;

* Regras de negócio no banco de dados: quais as vantagens e desvantagens? Stack Overflow em Português. Disponível em: https://pt.stackoverflow.com/questions/15739/regras-de-neg%C3%B3cio-no-banco-de-dados-quais-as-vantagens-e-desvantagens.

 ## 4. Conclusões

O grupo conclui que o sucesso de um sistema depende de aplicar a regra certa no lugar certo. O Banco de Dados não é apenas um local de armazenamento, mas a fundação que deve garantir a integridade de domínio, de chaves e a execução segura via transações ACID. Por outro lado, a Aplicação deve atuar como o maestro operacional, cuidando de validações dinâmicas e cálculos voláteis. Centralizar tudo no banco causa lentidão e engessamento; centralizar tudo na aplicação resulta em dados órfãos e corrompidos.

**Link do Repositório Git:** https://github.com/jcastrosousa/atividade-bd-g1.git
