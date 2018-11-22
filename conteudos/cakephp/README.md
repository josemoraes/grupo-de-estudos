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
4. A requisição e a resposta são processados através do PSR-7 ([PHP Standards Recommendation](https://pt.stackoverflow.com/questions/32295/o-que-significa-psr)) Middleware que sua aplicação utiliza. Normalmente isso inclui captura de erros e roteamento.
5. Se nenhuma resposta for retornada do middleware e a requisição contiver informações de rota, um Controller e uma action são acionados.
6. A action do Controller é chamada e o mesmo interage com os Models e Components requisitados.
7. O controller delega a responsabilidade de criar respostas à view, para assim gerar a saída de dados resultante do Model.
8. A View utiliza Helpers e Cells para gerar o corpo e cabeçalho das respostas.
9. A resposta é enviada de volta através do Middleware.
10. O HttpServer emite a resposta para o servidor web.




composer self-update && composer create-project --prefer-dist cakephp/app cms