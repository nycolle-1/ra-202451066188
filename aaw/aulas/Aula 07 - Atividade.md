# HANDOUT — AULA 07

## Caça às Vulnerabilidades

*Revisão de segurança de uma API .NET — Arquitetura de Aplicações Web*

## 🎯 MISSÃO

Vocês são a dupla de revisores de segurança da empresa. Os 4 trechos abaixo são da MESMA API, prestes a ir para produção. Para CADA card:

- Descrevam a falha com as próprias palavras (não precisa do nome técnico ainda)
- Estimem o dano possível se isso chegar à produção
- Proponham a correção

*⏱️ Tempo: 30 minutos  |  👥 Formato: em duplas  |  Todo o código é fictício e roda apenas no laboratório.*

> **Nomes:** ____________________   **Turma:** ____________________   **Data:** ___ / ___ / ______

## VULNERABILIDADE 01 — A busca de clientes- pronta

> `GET /api/clientes/buscar?nome=...`

Endpoint de busca usado pela tela de atendimento. O parâmetro nome vem direto da caixa de busca do site.

```text
 1  [HttpGet("buscar")]
 2  public IActionResult Buscar(string nome)
 3  {
 4      var sql = "SELECT * FROM Clientes WHERE Nome = '"
 5                + nome + "'";
 6      var clientes = _db.Clientes.FromSqlRaw(sql).ToList();
 7      return Ok(clientes);
 8  }
```

**Sua análise:**

1. Qual é a falha?
Injection - Dados estão sendo usado como o codigo. O nome do cliente vim diretamente na caixa de busca do site por ser informaçõe sensivel.

2. Qual o dano possível em produção?
Falta de segurança com o nome do cliente. Se tornando simples algum teceiro má intencionado consigua obter os dados sensiveis.

3. Como corrigir?
```text
 1  [HttpGet("buscar")]
 2  public IActionResult Buscar(string nome)
 3  {
        _db.Clientes.FromSqlInterpolated(
        $"SELECT * FROM Clientes" +
        $" WHERE Nome = {nome}");
 7      return Ok(clientes);
 8  }
```


## VULNERABILIDADE 02 — A consulta de faturas - pronta

> `GET /api/faturas/{id}`

Endpoint usado pelo app para exibir a fatura do cartão. O usuário está autenticado quando chama esta rota.

```text
 1  [HttpGet("{id}")]
 2  public IActionResult GetFatura(int id)
 3  {
 4      var fatura = _db.Faturas.Find(id);
 5      if (fatura == null) return NotFound();
 6      return Ok(fatura);
 7  }
```

**Sua análise:**

1. Qual é a falha?
Broken Acesse Control -  não a uma checagem de indentidade se a fatura percetem ao usuario antes de retonar.

2. Qual o dano possível em produção?
Dados sensiveis dentro da fatura sejam visto, acontecer um erro e um cliente pagar uma fatura que não lhe pertence.

3. Como corrigir?
```text
 1  [HttpGet("{id}")]
 2  public IActionResult GetFatura(int id)
 3  {
     var fatura = _db.Faturas.Find(id);
   
    if (fatura == null) return NotFound();
    if (fatura.ClienteId != usuarioLogadoId)
        
        return Forbid();  // 403
    return Ok(fatura);
 
 7  }
```
Tambem tiramos a saida NotFound para não informar exatamente a resposta sem a verificação.Caso alguem má intencionado recebesse, saberia que poderia continuar tentando ate achar.

## VULNERABILIDADE 03 — A configuração do servidor pronto

> `Program.cs (roda igual em dev e em produção)`

Trecho de inicialização da API, idêntico em todos os ambientes. Este arquivo está versionado no Git da empresa.

```text
 1  public const string Conn =
 2      "Server=prod-db;Database=Banco;User=sa;" +
 3      "Password=Newton@2026!";
 4
 5  var app = WebApplication.CreateBuilder(args).Build();
 6  app.UseDeveloperExceptionPage();
 7  app.Run();
```

**Sua análise:**

1. Qual é a falha?
Misconfiguration - a senha esta exporta no github exporto para sempre. 

2. Qual o dano possível em produção?
Senha exporta.

3. Como corrigir?
```text

builder.Configuration
       .GetConnectionString("Default");
if (app.Environment.IsDevelopment())
    app.UseDeveloperExceptionPage();
 4
 5  var app = WebApplication.CreateBuilder(args).Build();
 6  app.UseDeveloperExceptionPage();
 7  app.Run();
```

## VULNERABILIDADE 04 — A atualização de perfil

> `PUT /api/usuarios/{id}`

Endpoint que o app chama quando o usuário edita o próprio perfil. O corpo da requisição é o JSON enviado pelo cliente.

```text
 1  public class UsuarioUpdate
 2  {
 3      public string Nome  { get; set; }
 4      public string Email { get; set; }
 5      public string Role  { get; set; }   // "user" | "admin"
 6  }
 7
 8  [HttpPut("{id}")]
 9  public IActionResult Atualizar(int id, UsuarioUpdate dto)
10  {
11      _repo.AtualizarTudo(id, dto);
12      return NoContent();
13  }
```

**Sua análise:**

1. Qual é a falha?
mass assingnment - Confiar no usuario e da acesso "irrestrito". 

2. Qual o dano possível em produção?
Erros humano acontecem por inocencia mas tambem por malicia. Alguem poderia mudar o acessso do seu usuario para admistrador.

3. Como corrigir?

```text
 1  public class UsuarioUpdate
 2  {
 3      public string Nome  { get; set; }
 4      public string Email { get; set; }
 5      "
 6  }  
 7
 8  [HttpPut("{id}")]
 9  public IActionResult Atualizar(int id, UsuarioUpdate dto)
10  {
11      _repo.AtualizarTudo(id, dto);
12      return NoContent();
13  }

## DESAFIO peruntar qual é

1. Qual das 4 falhas um scanner automático de código teria MAIS dificuldade de encontrar? Por quê?

