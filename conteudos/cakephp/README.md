# CakePHP - Uma breve introdução

CakePHP é um framework baseado em PHP que tem o intutito de otimizar o desenvolvimento e entrega aplicações de baixa ou alta robustez em tempo ágil.
Ele possui convenções de nomenclaturas de classes, arquivos, base de dados, etc. Essa abordagem permite horizontalização do conhecimento e embora demore um tempo para assimilá-las, a curva de aprendizagem é baixa e o uso dessas convenções permite ainda reduzir configurações demasiadas. 

>Este tutorial é uma adaptação e replica o trabalho da página oficial do [CakePHP](https://book.cakephp.org/3.0/en/index.html).

## Convenções - Principais Conceitos

### Convenções para Controllers

Os nomes das classes de Controllers são pluralizados, CamelCased, e terminam em Controller. `PeopleController` e `LatestArticlesController` são exemplos de nomes convencionais para controllers.

Métodos públicos nos Controllers são frequentemente referenciados como ‘actions’ acessíveis através de um navegador web. Por exemplo, o `/articles/view` mapeia para o método `view()` do `ArticlesController` sem nenhum esforço. Métodos privados ou protegidos não podem ser acessados pelo roteamento.

#### Considerações de URL para nomes de Controller

Como você acabou de ver, controllers singulares mapeiam facilmente um caminho simples, todo em minúsculo. Por exemplo, `ApplesController` (o qual deveria ser definido no arquivo de nome ‘ApplesController.php’) é acessado por `http://example.com/apples`.

Controllers com múltiplas palavras podem estar em qualquer forma ‘flexionada’ igual ao nome do controller, então:

* /redApples
* /RedApples
* /Red_apples
* /red_apples

Todos resolverão para o index do controller RedApples. Porém, a forma correta é que suas URLs sejam minúsculas e separadas por sublinhado, portanto `/red_apples/go_pick` é a forma correta de acessar a action `RedApplesController::go_pick`.

#### Convenções para Models e Databases

Os nomes de classe de Tables são pluralizadas e CamelCased. `People`, `BigPeople`, and `ReallyBigPeople` são todos exemplos convencionais de models.
Os nomes de Tables correspondentes aos models do CakePHP são pluralizadas e separadas por sublinhado. As tables sublinhadas para os models mencionados acima seriam `people`, `big_people`, e `really_big_people`, respectivamente.
Chaves estrangeiras nos relacionamentos *hasMany*, *belongsTo* ou *hasOne* são reconhecidas por padrão como o nome (singular) da table relacionada seguida por _id. Então se `Bakers hasMany Cakes`, a table `cakes` irá referenciar-se para a table `bakers` através da chave estrangeira `baker_id`.

#### Convenções para Views

Arquivos de template views são nomeadas seguindo as funções que a exibem do controller, separadas por sublinhado. A função `getReady()` da classe `PeopleController` buscará por um template view em `src/Template/People/get_ready.ctp`. O padrão é `src/Template/Controller/underscored_function_name.ctp`.

> Para mais informações sobre convenções em PHP, consulte [aqui](https://book.cakephp.org/3.0/pt/intro/conventions.html).

## Camadas do CakePHP

O CakePHP se divide, em linhas gerais, em três camadas: Model-View-Controller (acho que já vi isso em algum lugar...). Cada camada possui um papel na arquitetura que inferem no resultado geral baixa coesão, o que permite maior facilidade na manutenção e evolução dos projetos.

### A camada Model
A camada Model representa a parte da sua aplicação que implementa a lógica de negócio. Ela é responsável por recuperar dados e convertê-los nos conceitos significativos primários na sua aplicação. Isto inclui processar, validar, associar ou qualquer outra tarefa relacionada à manipulação de dados.

No caso de uma rede social, a camada Model deveria tomar cuidado de tarefas como salvar os dados do usuário, salvar as associações entre amigos, salvar e recuperar fotos de usuários, localizar sugestões para novos amigos, etc. Os objetos de modelo podem ser pensados como “Friend”, “User”, “Comment”, ou “Photo”. Se nós quiséssemos carregar alguns dados da nossa tabela users poderiamos fazer:

```php
use Cake\ORM\TableRegistry;

$users = TableRegistry::get('Users');
$query = $users->find();
foreach ($query as $row) {
    echo $row->username;
}
```
Você pode notar que não precisamos escrever nenhum código antes de podermos começar a trabalhar com nossos dados. Por usar convenções, o CakePHP irá utilizar classes padrão para tabelas e entidades ainda não definidas.

Se nós quiséssemos criar um usuário e salvá-lo (com validação) fariamos algo assim:

```php
use Cake\ORM\TableRegistry;

$users = TableRegistry::get('Users');
$user = $users->newEntity(['email' => 'mark@example.com']);
$users->save($user);
```

### A camada View
A View renderiza uma apresentação de dados modelados. Estando separada dos objetos da Model, é responsável por utilizar a informação que tem disponível para produzir qualquer interface de apresentação que a sua aplicação possa precisar.

Por exemplo, a view pode usar dados da model para renderizar uma página HTML que os conhtenha, ou um resultado formatado como XML:

```php
// No arquivo view, nós renderizaremos um 'elemento' para cada usuário.
<?php foreach ($users as $user): ?>
    <div class="user">
        <?= $this->element('user', ['user' => $user]) ?>
    </div>
<?php endforeach; ?>
```
A camada View não está limitada somente a HTML ou apresentação textual dos dados. Ela pode ser usada para entregar formatos de dado comuns como JSON, XML, e através de uma arquitetura encaixável qualquer outro formato que você venha precisar.

### A camada Controller

A camada Controller manipula requisições dos usuários. É responsável por renderizar uma resposta com o auxílio de ambas as camadas, Model e View respectivamente.

Um controller pode ser visto como um gerente que certifica-se que todos os recursos necessários para completar uma tarefa sejam delegados aos trabalhadores corretos. Ele aguarda por petições dos clientes, checa suas validades de acordo com autenticação ou regras de autorização, delega requisições ou processamento de dados da camada Model, selecciona o tipo de dados de apresentação que os clientes estão aceitando, e finalmente delega o processo de renderização para a camada View. Um exemplo de controller para registro de usuário seria:

```php
public function add()
{
    $user = $this->Users->newEntity();
    if ($this->request->is('post')) {
        $user = $this->Users->patchEntity($user, $this->request->getData());
        if ($this->Users->save($user, ['validate' => 'registration'])) {
            $this->Flash->success(__('Você está registrado.'));
        } else {
            $this->Flash->error(__('Houve algum problema.'));
        }
    }
    $this->set('user', $user);
}
```

## Ciclo de Requisições do CakePHP

O cíclo de requisição do CakePHP começa com a solicitação de uma página ou recurso da sua aplicação, seguindo a cadência abaixo:

![Ciclo de Requisições do CakePHP](/conteudos/cakephp/img/ciclo-cake.png)

1. As regras de reescrita do servidor encaminham a requisição para webroot/index.php.
2. Sua aplicação é carregada e vinculada a um `HttpServer`.
3. O middleware da sua aplicação é inicializado.
4. A requisição e a resposta são processados através do PSR-7 Middleware ([PHP Standards Recommendation](https://pt.stackoverflow.com/questions/32295/o-que-significa-psr)) que sua aplicação utiliza. Normalmente isso inclui captura de erros e roteamento.
5. Se nenhuma resposta for retornada do middleware e a requisição contiver informações de rota, um Controller e uma action são acionados.
6. A action do Controller é chamada e o mesmo interage com os Models e Components requisitados.
7. O controller delega a responsabilidade de criar respostas à view, para assim gerar a saída de dados resultante do Model.
8. A View utiliza Helpers e Cells para gerar o corpo e cabeçalho das respostas.
9. A resposta é enviada de volta através do Middleware.
10. O HttpServer emite a resposta para o servidor web.


# Prática - Como desenvolver o básico de um blog

Antes de iniciarmos, devemos nos certificar de que seu ambiente de desenvolvimento atende aos seguintes pré-requisitos:

* Apache com mod_rewrite habilitado;
* Banco de dados MySQL instalado; e
* PHP 5.6.0 ou superior, configurado;
* Extensões PHP instaladas e habilitadas: mbstring; intl; e simplexml;
* PHP-cli configurado globalmente;
* Composer instalado globalmente.

> Como nosso grupo possui maioria de estudantes com computadores com o SO Windows, será considerado que o ambiente foi preparado com o Xampp.

Para verificar se está tudo certo, vá ao diretório do xampp `./htdocs`; crie um diretório `ambiente` e dentro dessa pasta crie um arquivo `index.php` com o seguinte código:

```php

<?php echo phpinfo();

```
Acesse `localhost/ambiente`; Isso permitirá que você veja quais são as extensões habilitadas para seu ambiente.


## Mãos ao código

Abra o terminal e navegue até `./htdocs/`; Em seguida, digite o seguinte comando: `composer self-update && composer create-project --prefer-dist cakephp/app blog`.
Esse comando solicitará ao composer para instalar o CakePHP no diretório `./blog`.

Após o composer concluir a instalação, acesse `localhost/blog`. Você deve ver a seguinte tela:

![CakePHP - Instalação](/conteudos/cakephp/img/instalacao.png)

O próximo passo é configurar o banco de dados.

- Crie um banco de dados com o nome `blog` usando o SGBD de preferência;
- Em seguida no diretório do seu projeto (`./htdocs/blog`), vá ao diretório `./app` e edite o `DataSource` no arquivo `app.php`, conforme o código abaixo:

```php
'Datasources' => [
        'default' => [
            'className' => 'Cake\Database\Connection',
            'driver' => 'Cake\Database\Driver\Mysql',
            'persistent' => false,
            'host' => 'localhost',
            /*
             * CakePHP will use the default DB port based on the driver selected
             * MySQL on MAMP uses port 8889, MAMP users will want to uncomment
             * the following line and set the port accordingly
             */
            //'port' => 'non_standard_port_number',
            'username' => 'root',
            'password' => '',
            'database' => 'blog',
            /*
             * You do not need to set this flag to use full utf-8 encoding (internal default since CakePHP 3.6).
             */
            //'encoding' => 'utf8mb4',
            'timezone' => 'UTC',
            'flags' => [],
            'cacheMetadata' => true,
            'log' => false,

            /**
             * Set identifier quoting to true if you are using reserved words or
             * special characters in your table or column names. Enabling this
             * setting will result in queries built using the Query Builder having
             * identifiers quoted when creating SQL. It should be noted that this
             * decreases performance because each query needs to be traversed and
             * manipulated before being executed.
             */
            'quoteIdentifiers' => false,

            /**
             * During development, if using MySQL < 5.6, uncommenting the
             * following line could boost the speed at which schema metadata is
             * fetched from the database. It can also be set directly with the
             * mysql configuration directive 'innodb_stats_on_metadata = 0'
             * which is the recommended value in production environments
             */
            //'init' => ['SET GLOBAL innodb_stats_on_metadata = 0'],

            'url' => env('DATABASE_URL', null),
        ],
```

Feito isso, atualize a página inicial de sua aplicação e veja se a configuração do banco é resolvida, conforme a imagem abaixo:

![CakePHP - Banco Configurado](/conteudos/cakephp/img/banco-configurado.png)

Se estiver tudo certo, significa que podemos avançar para a próxima etapa. A criação da tabela por meio de Migrações.

## Migrações com CakePHP

O CakePHP implementa um algoritmo de migrações capaz de abstrair a camada física do banco de dados para uma camada lógica interoperável. São conhecidas aqui como [*Migrations*](https://book.cakephp.org/3.0/en/phinx/migrations.html).
Para representarmos uma entidade `Articles (id:integer, title:string:limit=50, body:string, created:datetime, modified:datetime)` utilizando esse mecanismo podemos seguir as seguintes etapas:

- Com o uso do [Gerador de Código](https://book.cakephp.org/3.0/pt/bake/usage.html) criaremos o arquivo de base da migração. Para isso, acesse por linha de comando o diretório `./blog/bin` e execute o seguinte comando: `cake bake migration CreateArticles`;
	- Acesse a pasta `./config/Migrations` e veja se existe um arquivo com nome <<SUFIXO>>_CreateArticles.php;
	- Caso exista, insira nele o seguinte código:

```php
<?php
use Migrations\AbstractMigration;

class CreateArticles extends AbstractMigration
{
    /**
     * Change Method.
     *
     * More information on this method is available here:
     * http://docs.phinx.org/en/latest/migrations.html#the-change-method
     * @return void
     */
    public function change()
    {
    	# Quando é utilizado a opção id=false, estou configurando um ID diferente do padrão criado automaticamente pelo framework
        $table = $this->table('articles', ['id'=>false]);
        $table
        ->addColumn('id', 'integer', ['autoIncrement'=>true])
        ->addColumn('title', 'string', ['limit'=>50])
        ->addColumn('body', 'string')
        ->addColumn('created', 'datetime')
        ->addColumn('modified', 'datetime')
        ->addPrimaryKey('id')
        ->create();
    }
}


```

- Em seguida, abra o terminal no diretório `./blog/bin` e execute o comando: `cake migrations migrate`.
	- Esse comando vai criar a tabela `Articles` no banco.

## Criando o CRUD usando o Gerador de Código

Considerando que utilizamos as convenções estabelecidas para o CakePHP, o uso do Gerador de Código será um verdadeiro ganho de tempo em termos de definição de telas padrões. Para utilizarmos, acesse a pasta `./blog/bin` e execute o comando: `cake bake all Articles`.
Com esse comando, criamos a camada Model-View-Controller para a entidade Articles.
Para testar acesse: `localhost/blog/articles`; veja se a tela exibida é semelhante ao da Figura a seguir:

![CakePHP - Articles, index](/conteudos/cakephp/img/articles-index.png)

A partir daí, as operações de criação, edição, listagem e exclusão estão funcionando para essa entidade.