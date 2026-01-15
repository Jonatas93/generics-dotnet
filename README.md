# Generics em .NET 🎯

## O que são Generics?

Generics são um recurso poderoso do C# e .NET que permite criar classes, métodos, interfaces e delegates que trabalham com **tipos parametrizados**. Em vez de escrever código específico para cada tipo de dado, você escreve código **reutilizável** que funciona com qualquer tipo.

### Analogia do Mundo Real

Imagine uma caixa organizadora:
- **Sem Generics**: Você precisa de uma caixa específica para guardar livros, outra para guardar sapatos, outra para guardar brinquedos...
- **Com Generics**: Você tem uma caixa universal que pode guardar **qualquer coisa**, mas quando você declara o que vai guardar nela, ela só aceita aquele tipo específico.

## Por que usar Generics?

### ✅ Vantagens

1. **Reutilização de Código**: Escreva uma vez, use com vários tipos
2. **Type Safety**: O compilador garante que você está usando os tipos corretos
3. **Performance**: Evita boxing/unboxing de tipos valor
4. **Legibilidade**: Código mais limpo e expressivo

### ❌ Sem Generics (Código Duplicado)

```csharp
// Precisamos criar uma classe para cada tipo 😢
public class RepositorioCliente
{
    public void Adicionar(Cliente item) { }
    public Cliente ObterPorId(int id) { return null; }
}

public class RepositorioProduto
{
    public void Adicionar(Produto item) { }
    public Produto ObterPorId(int id) { return null; }
}

// E assim por diante... muito código repetido!
```

### ✅ Com Generics (Código Reutilizável)

```csharp
// Uma única classe genérica serve para TODOS os tipos! 🎉
public class Repositorio<T>
{
    public void Adicionar(T item) { }
    public T ObterPorId(int id) { return default; }
}

// Uso
var repoClientes = new Repositorio<Cliente>();
var repoProdutos = new Repositorio<Produto>();
var repoFornecedores = new Repositorio<Fornecedor>();
```

## Sintaxe Básica

### Classes Genéricas

```csharp
public class Caixa<T>
{
    private T _conteudo;
    
    public void Guardar(T item)
    {
        _conteudo = item;
    }
    
    public T Recuperar()
    {
        return _conteudo;
    }
}

// Uso
var caixaDeTexto = new Caixa<string>();
caixaDeTexto.Guardar("Olá, Generics!");
string texto = caixaDeTexto.Recuperar(); // "Olá, Generics!"

var caixaDeNumeros = new Caixa<int>();
caixaDeNumeros.Guardar(42);
int numero = caixaDeNumeros.Recuperar(); // 42
```

### Métodos Genéricos

```csharp
public class Utilidades
{
    // Método genérico que troca dois valores
    public static void Trocar<T>(ref T a, ref T b)
    {
        T temp = a;
        a = b;
        b = temp;
    }
}

// Uso
int x = 10, y = 20;
Utilidades.Trocar(ref x, ref y); // x = 20, y = 10

string nome1 = "João", nome2 = "Maria";
Utilidades.Trocar(ref nome1, ref nome2); // nome1 = "Maria", nome2 = "João"
```

### Interfaces Genéricas

```csharp
public interface IRepositorio<T>
{
    void Adicionar(T item);
    T ObterPorId(int id);
    List<T> ObterTodos();
}

public class RepositorioEmMemoria<T> : IRepositorio<T>
{
    private List<T> _itens = new List<T>();
    
    public void Adicionar(T item)
    {
        _itens.Add(item);
    }
    
    public T ObterPorId(int id)
    {
        // implementação
        return default;
    }
    
    public List<T> ObterTodos()
    {
        return _itens;
    }
}
```

## Constraints (Restrições de Tipo) 🔒

Constraints permitem **restringir** quais tipos podem ser usados com seus Generics. Isso dá mais controle e permite acessar membros específicos do tipo genérico.

### `where T : class` - Tipo Referência

Garante que `T` deve ser uma **classe** (tipo referência), não um tipo valor.

```csharp
public class GerenciadorDeEntidades<T> where T : class
{
    private T _entidade;
    
    public bool EntidadeEhNula()
    {
        // Podemos verificar null porque T é garantidamente uma classe
        return _entidade == null;
    }
}

// ✅ VÁLIDO - Classes (tipos referência)
var gerenciador1 = new GerenciadorDeEntidades<Cliente>();
var gerenciador2 = new GerenciadorDeEntidades<string>();
var gerenciador3 = new GerenciadorDeEntidades<Produto>();

// ❌ ERRO DE COMPILAÇÃO - Tipos valor
var gerenciador4 = new GerenciadorDeEntidades<int>();      // ERRO!
var gerenciador5 = new GerenciadorDeEntidades<DateTime>(); // ERRO!
var gerenciador6 = new GerenciadorDeEntidades<bool>();     // ERRO!
```

**Quando usar `where T : class`:**
- Quando você precisa verificar se o objeto é `null`
- Quando trabalha com herança de classes
- Quando quer garantir comportamento de tipo referência

### `where T : struct` - Tipo Valor

Garante que `T` deve ser um **tipo valor** (struct, int, DateTime, etc.).

```csharp
public class CalculadoraGenerica<T> where T : struct
{
    public T? ValorOpcional { get; set; } // Nullable só funciona com struct
}

// ✅ VÁLIDO - Tipos valor
var calc1 = new CalculadoraGenerica<int>();
var calc2 = new CalculadoraGenerica<DateTime>();
var calc3 = new CalculadoraGenerica<decimal>();

// ❌ ERRO DE COMPILAÇÃO - Classes
var calc4 = new CalculadoraGenerica<string>();  // ERRO!
var calc5 = new CalculadoraGenerica<Cliente>(); // ERRO!
```

### `where T : new()` - Construtor Sem Parâmetros

Garante que `T` tem um construtor público sem parâmetros.

```csharp
public class Fabrica<T> where T : new()
{
    public T CriarInstancia()
    {
        return new T(); // Podemos criar instâncias!
    }
}

public class Produto
{
    public Produto() { } // Construtor sem parâmetros - OK!
}

public class Cliente
{
    public Cliente(string nome) { } // Só tem construtor COM parâmetro - ERRO!
}

// ✅ VÁLIDO
var fabrica1 = new Fabrica<Produto>();
var novoProduto = fabrica1.CriarInstancia(); // Funciona!

// ❌ ERRO DE COMPILAÇÃO
var fabrica2 = new Fabrica<Cliente>(); // ERRO! Cliente não tem construtor sem parâmetros
```

### `where T : MinhaClasse` - Herança de Classe Específica

Garante que `T` herda de uma classe específica.

```csharp
public abstract class Entidade
{
    public int Id { get; set; }
    public DateTime DataCriacao { get; set; }
}

public class Repositorio<T> where T : Entidade
{
    public void Salvar(T entidade)
    {
        // Podemos acessar propriedades de Entidade!
        Console.WriteLine($"Salvando entidade com ID: {entidade.Id}");
        entidade.DataCriacao = DateTime.Now;
    }
}

public class Cliente : Entidade
{
    public string Nome { get; set; }
}

public class Produto : Entidade
{
    public string Descricao { get; set; }
}

// ✅ VÁLIDO - Cliente e Produto herdam de Entidade
var repoClientes = new Repositorio<Cliente>();
var repoProdutos = new Repositorio<Produto>();

// ❌ ERRO - string não herda de Entidade
var repoString = new Repositorio<string>(); // ERRO!
```

### `where T : IMinhaInterface` - Implementação de Interface

Garante que `T` implementa uma interface específica.

```csharp
public interface ISerializavel
{
    string Serializar();
    void Desserializar(string dados);
}

public class GerenciadorDeArquivos<T> where T : ISerializavel
{
    public void SalvarEmArquivo(T objeto, string caminho)
    {
        // Podemos chamar métodos da interface!
        string dados = objeto.Serializar();
        File.WriteAllText(caminho, dados);
    }
}

public class Configuracao : ISerializavel
{
    public string Serializar() => "...";
    public void Desserializar(string dados) { }
}

// ✅ VÁLIDO
var gerenciador = new GerenciadorDeArquivos<Configuracao>();
```

### Múltiplas Constraints

Você pode combinar várias restrições!

```csharp
public class RepositorioAvancado<T> 
    where T : Entidade, ISerializavel, new()
{
    // T deve:
    // 1. Herdar de Entidade
    // 2. Implementar ISerializavel
    // 3. Ter construtor sem parâmetros
}
```

### Tabela Resumo de Constraints

| Constraint | Descrição | Exemplo |
|------------|-----------|---------|
| `where T : class` | T deve ser tipo referência (classe) | `string`, `Cliente`, `List<int>` |
| `where T : struct` | T deve ser tipo valor | `int`, `DateTime`, `bool` |
| `where T : new()` | T deve ter construtor sem parâmetros | Qualquer classe com `public T()` |
| `where T : MinhaClasse` | T deve herdar de MinhaClasse | Classes derivadas |
| `where T : IMinhaInterface` | T deve implementar a interface | Classes que implementam |
| `where T : U` | T deve ser ou derivar de U | Para múltiplos parâmetros de tipo |

## Exemplos do Mundo Real

### Sistema de Cache Genérico

```csharp
public class CacheGenerico<TChave, TValor> where TValor : class
{
    private Dictionary<TChave, TValor> _cache = new Dictionary<TChave, TValor>();
    
    public void Adicionar(TChave chave, TValor valor)
    {
        _cache[chave] = valor;
    }
    
    public TValor Obter(TChave chave)
    {
        return _cache.ContainsKey(chave) ? _cache[chave] : null;
    }
    
    public bool Existe(TChave chave)
    {
        return _cache.ContainsKey(chave);
    }
}

// Uso
var cacheUsuarios = new CacheGenerico<int, Usuario>();
cacheUsuarios.Adicionar(1, new Usuario { Nome = "João" });
Usuario usuario = cacheUsuarios.Obter(1);
```

### Serviço Genérico de CRUD

```csharp
public interface IEntidadeComId
{
    int Id { get; set; }
}

public class ServicoCrud<T> where T : IEntidadeComId, new()
{
    private List<T> _dados = new List<T>();
    
    public void Criar(T entidade)
    {
        entidade.Id = _dados.Count + 1;
        _dados.Add(entidade);
    }
    
    public T Ler(int id)
    {
        return _dados.FirstOrDefault(x => x.Id == id);
    }
    
    public void Atualizar(T entidade)
    {
        var indice = _dados.FindIndex(x => x.Id == entidade.Id);
        if (indice >= 0)
            _dados[indice] = entidade;
    }
    
    public void Deletar(int id)
    {
        _dados.RemoveAll(x => x.Id == id);
    }
}
```

### Processador de Mensagens

```csharp
public interface IMensagem
{
    DateTime DataEnvio { get; set; }
}

public class ProcessadorDeMensagens<T> where T : IMensagem
{
    public void Processar(T mensagem)
    {
        Console.WriteLine($"Processando mensagem de {mensagem.DataEnvio}");
        // lógica específica
    }
    
    public List<T> FiltrarPorPeriodo(List<T> mensagens, DateTime inicio, DateTime fim)
    {
        return mensagens.Where(m => m.DataEnvio >= inicio && m.DataEnvio <= fim).ToList();
    }
}
```

## Generics na BCL (Base Class Library) do .NET

O .NET usa Generics extensivamente:

```csharp
// Coleções Genéricas
List<int> numeros = new List<int>();
Dictionary<string, Cliente> clientes = new Dictionary<string, Cliente>();
Queue<Pedido> filaPedidos = new Queue<Pedido>();
Stack<Operacao> pilhaOperacoes = new Stack<Operacao>();

// Nullable Types
int? numeroOpcional = null;
DateTime? dataOpcional = null;

// Task e Async
Task<string> tarefa = ObterDadosAsync();
string resultado = await tarefa;

// LINQ
IEnumerable<Cliente> clientesAtivos = clientes.Where(c => c.Ativo);
```

## Boas Práticas

1. **Use nomes descritivos**: `T` para tipo único, `TKey` e `TValue` para pares, `TEntity` para entidades
2. **Aplique constraints quando necessário**: Ajuda na legibilidade e evita erros
3. **Prefira Generics a `object`**: Melhor performance e type safety
4. **Documente as constraints**: Deixe claro o que é esperado do tipo genérico

## Conclusão

Generics são fundamentais para escrever código .NET moderno, reutilizável e type-safe. Dominar este conceito é essencial para qualquer desenvolvedor C#!

---

**Recursos Adicionais:**
- [Documentação Oficial Microsoft - Generics](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/generics)
- [C# Programming Guide - Generic Constraints](https://learn.microsoft.com/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)

---

⭐ Se este conteúdo foi útil, deixe uma estrela no repositório!
