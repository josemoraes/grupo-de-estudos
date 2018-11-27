# Tarefas

Vimos na seção principal desse conteúdo que o CakePHP é capaz de otimizar o processo de entregas de aplicações de baixa e alta complexidade. Recursos como Migrações e Gerador de Código, garantem um aumento expressivo na velocidade de desenvolvimento.
Com o objetivo de atestar os conhecimentos passados anteriormente, serão sugeridos 2 tipos de exercícios. A realização deles deve ser feita em pares.

## Sugestão 01

Joaquim é um homem muito organizado, e como ele é seu amigo, te pediu que desenvolvesse um gerenciador de finanças pessoais.

### Premissas

1. Devo ser capaz de cadastrar despesas e receitas. Tanto despesa quanto receita devem possuir: ORIGEM (e.g. Compra de um lanchão no Burger King); o VALOR; a DATA de quando a movimentação aconteceu (tanto da receita, quanto da despesa aconteceu);
2. Ele deve ter uma página pessoal, aonde ele poderá verificar a lista de receitas e despesas que gerou com um balanço da relação despesas/receitas.

> Bônus: Quem quiser se arriscar, pode criar um filtro de data para a listagem do relatório.

### Restrição

1. Não é necessário criar mecanismo de cadastro e autenticação de usuário, pois será um sistema utilizado apenas por ele.



-------------------------------------------------------------------------------------------------------------------


## Sugestão 02

Ash é um mestre pokémon, e como vocês são amigos de longa data ele te pediu uma ajuda para criar uma pokedéx na web para que outros mestres pokémon possam compartilhar suas conquistas.

### Premissas

1. Cadastrar usuários com: Nome; Registro de mestre pokémon (tem 8 dígitos alfanuméricos);
2. Cadastrar insígnias com: Nome; Data de conquista da insígnia;
3. Cadastrar pokémon com: Nome; Data de captura; Peso; Altura; e quais são seus principais ataques;

### Restrição
1. Não é necessário uso de máscaras nos campos;
2. Não é necessário criar mecanismo de autenticação de usuário; 
3. No cadastro de informações será necessário apenas indicar qual é o usuário relacionado.


# Alguns links que poderão ajudar

* [Gerador de Código](https://book.cakephp.org/3.0/en/bake/usage.html)
* [Migrations](https://book.cakephp.org/3.0/en/migrations.html)
* [Convenções](https://book.cakephp.org/3.0/en/intro/conventions.html)