# Interface, Service, Controller e Endpoint: Buscar Autor por ID do Livro

[← Voltar](https://github.com/JosiTubaroski/WEB-API-com-.NET-8-e-SQL-Server)

![Fluxo Controller, Interface e Service](https://github.com/JosiTubaroski/Controllers_Services/blob/main/img/01_Fx_Controller_Interface_Service_2.jpg)

Dando sequência ao endpoint de busca de autor por ID, esta etapa implementa uma variação da mesma ideia: em vez de buscar o autor pelo seu próprio ID, a busca é feita a partir do **ID do livro** — ou seja, "para este livro, quem é o autor?". É um bom exemplo de como pequenas variações de um mesmo caso de uso (buscar um autor) podem exigir métodos diferentes, dependendo de qual informação o consumidor da API já tem disponível.

## 1. Implementando o Método no AutorService.cs

O método `BuscarAutorPorIdLivro`, previamente declarado na interface `IAutorInterface`, é implementado no `AutorService.cs`.

![Método BuscarAutorPorIdLivro no AutorService](https://github.com/JosiTubaroski/Listar_Autor_Por_IDLivro/blob/main/img/05_BuscarAutorPorIDLivro.png)

### Explicação do Código

Diferente do `BuscarAutorPorId` (que consulta diretamente a tabela `Autores` pelo ID do autor), este método precisa relacionar duas tabelas: `Livros` e `Autores`. A lógica esperada é:

1. Buscar o **livro** correspondente ao `idLivro` informado.
2. A partir desse livro, obter o **autor** relacionado a ele (via propriedade de navegação `Autor`, definida na `LivroModel`).
3. Retornar os dados do autor dentro do `ResponseModel<AutorModel>`, seguindo o mesmo padrão de resposta dos demais métodos: dados encontrados, "nenhum registro localizado" ou erro tratado no `catch`.

Esse é um exemplo prático de como o Entity Framework Core simplifica consultas que envolvem relacionamentos entre tabelas — em vez de escrever um JOIN em SQL manualmente, é possível navegar diretamente entre as entidades relacionadas (`Livro.Autor`), já que o relacionamento foi definido nas Models.

## 2. Incluindo o Método no AutorController.cs

Com a lógica implementada no Service, o método é exposto como um novo endpoint no `AutorController.cs`.

![BuscarAutorPorIdLivro no AutorController](https://github.com/JosiTubaroski/Listar_Autor_Por_IDLivro/blob/main/img/06_AutorController_IdLivros.png)

Seguindo o padrão já utilizado nos demais endpoints:

- O método é decorado com `[HttpGet(...)]`, definindo a rota (por exemplo, recebendo `idLivro` como parâmetro de rota).
- Recebe o `idLivro` como parâmetro.
- Delega a lógica de busca para `_autorInterface.BuscarAutorPorIdLivro(idLivro)`.
- Retorna o resultado com `Ok(...)`, mantendo a resposta padronizada em `ResponseModel<T>`.

## 3. Testando o Método

Com o endpoint implementado, o projeto é executado e o método é testado através do Swagger, informando o ID de um livro existente.

![Testando o Método BuscarAutorPorIdLivro](https://github.com/JosiTubaroski/Listar_Autor_Por_IDLivro/blob/main/img/07_Buscar_AutorIdLivro.png)

Ao informar um `idLivro` válido e executar a requisição, a API retorna os dados do autor relacionado àquele livro. Se o ID do livro não existir na base de dados, o comportamento esperado — seguindo o padrão já estabelecido nos outros métodos — é retornar `status 200` com `dados` vazio e uma mensagem informando que nenhum registro foi localizado.

## Resumo do Fluxo

1. **Service** — implementa `BuscarAutorPorIdLivro`, localizando primeiro o livro e, a partir dele, o autor relacionado.
2. **Controller** — expõe esse método como um endpoint HTTP GET, recebendo o ID do livro via rota.
3. **Teste** — validação via Swagger, informando o ID de um livro e conferindo se o autor correto é retornado.

## Ideias e Conclusões

- **Um mesmo dado, diferentes portas de entrada.** Os três métodos criados até aqui (`ListarAutores`, `BuscarAutorPorId` e `BuscarAutorPorIdLivro`) mostram como uma API bem desenhada oferece diferentes formas de acessar a mesma informação (o autor), de acordo com o que o consumidor da API já tem em mãos — às vezes um ID de autor, às vezes um ID de livro.
- **Navegação entre entidades relacionadas.** Esse endpoint reforça a importância de modelar corretamente os relacionamentos entre as Models (`LivroModel` → `AutorModel`). Quando o relacionamento está bem definido, o Entity Framework Core permite "atravessar" de uma entidade para outra sem a necessidade de escrever consultas SQL manuais.
- **Padrão de resposta consistente.** Mesmo mudando a lógica de busca, o método continua retornando um `ResponseModel<T>` com a mesma estrutura (`dados`, `mensagem`, `status`) — o que mantém a API previsível para quem a consome, independentemente do endpoint chamado.
- **Ponto de atenção para evoluir esse endpoint:** vale considerar o que a API deve retornar caso o `idLivro` exista, mas o livro não tenha um autor vinculado (registro nulo) — esse é um cenário diferente de "livro não encontrado", e pode merecer uma mensagem própria para facilitar o diagnóstico por quem consome a API.
