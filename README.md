<div align="center">
  
  <img src="https://capsule-render.vercel.app/api?type=waving&color=timeGradient&height=250&section=header&text=Spring%20Boot%20&%20System%20Design&fontSize=50&fontAlignY=38&desc=O%20Manual%20Definitivo&descAlignY=60&descAlign=50" alt="Banner" />

  <h1>🚀 Meu Framework de Desenvolvimento Backend</h1>
  <p><i>Um guia prático e opinativo de como eu desenvolvo APIs escaláveis, limpas e prontas para produção no Spring Boot.</i></p>

  <img src="https://img.shields.io/badge/Java-17+-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white" alt="Java" />
  <img src="https://img.shields.io/badge/Spring_Boot-3.x-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white" alt="Spring Boot" />
  <img src="https://img.shields.io/badge/Arquitetura-Layered-316192?style=for-the-badge" alt="Arquitetura" />
  <img src="https://img.shields.io/badge/Status-Em_Evolução-FFD700?style=for-the-badge" alt="Status" />

</div>

<br>

## 📌 Sobre o Projeto

Este repositório não é apenas código. É o meu **framework pessoal de desenvolvimento backend**, onde aplico:
- Boas práticas de mercado.
- Arquitetura limpa.
- Princípios sólidos de engenharia de software.
- Padrões usados em sistemas reais.

O objetivo aqui é mostrar **como eu penso, estruturo e desenvolvo software backend profissional**.

---

## 📑 Índice Interativo


- [1. Filosofia de Desenvolvimento 🧠](#1-filosofia-de-desenvolvimento-)
- [2. Estrutura do Projeto 📂](#2-estrutura-do-projeto-)
- [3. Setup Inicial 🛠️](#3-setup-inicial-️)
- [4. A Mão na Massa (Fluxo Completo) 💻](#4-a-mão-na-massa-fluxo-completo-)
- [5. Tratamento de Erros e Boas Práticas 🛡️](#5-tratamento-de-erros-e-boas-práticas-️)
- [6. Segurança (Spring Security & JWT) 🔒](#6-segurança-spring-security--jwt-)
- [7. Escalabilidade e System Design 🚀](#7-escalabilidade-e-system-design-)

---

## 🧠 1. Filosofia de Desenvolvimento

Antes de escrever a primeira linha de código, eu sigo regras claras. O objetivo é criar um código que seja **lido por humanos primeiro, e por máquinas depois**.

### 📐 Princípios Base
* **SOLID:** Foco em classes com responsabilidades únicas e baixo acoplamento.
* **DRY (Don't Repeat Yourself):** Reutilização inteligente de código.
* **KISS (Keep It Simple, Stupid):** Soluções simples para problemas complexos.

### 🧹 Clean Code (Código Limpo)
* **Nomenclatura Clara:** Nada de `x`, `y` ou `aux`. Se busca um usuário por ID, o método se chama `findUserById()`.
* **Métodos Pequenos:** Um método deve fazer **apenas uma coisa**. Passou de 20 linhas? Extraia a lógica.
* **Inglês como Padrão:** Código e commits em inglês para facilitar a integração global.
* **Comentários são a exceção:** O código deve ser autoexplicativo. Comente o *porquê* de uma regra complexa, não o *que* o código faz.

> [!WARNING]  
> **Regras de Ouro:**
> 1. **Nunca vaze a Entidade:** Entidades (`@Entity`) não chegam ao Controller. Use **DTOs** para entrada e saída.
> 2. **Erros Centralizados:** Sem `try-catch` espalhados. Use `@ControllerAdvice`.

---

## 📂 2. Estrutura do Projeto

Separação rígida de responsabilidades: **Controller ➝ Service ➝ Repository**.

```text
src/main/java/com/meuframework/app
├── config/       # Configurações globais e Setup (Segurança, CORS).
├── controller/   # Entradas HTTP e devolução de Respostas HTTP.
├── dto/          # Data Transfer Objects (Request e Response).
├── model/        # Entidades de domínio (Tabelas do banco com JPA).
├── repository/   # Camada de acesso a dados (JpaRepository).
├── service/      # Regras de negócio e validações.
├── exception/    # Erros customizados e GlobalExceptionHandler.
└── infra/        # Integrações externas (RabbitMQ, Kafka, AWS).
````

-----

## 🛠️ 3. Setup Inicial

### 🔧 Dependências Base

  * **Spring Web:** Traz o Tomcat e anotações REST.
  * **Spring Data JPA:** Abstrai o SQL e JDBC.
  * **Lombok:** Remove a necessidade de digitar Getters e Setters.
  * **Validation:** Valida DTOs de entrada.
  * **PostgreSQL Driver & Flyway:** Banco de dados e controle de versão de schemas.

### 📄 `application.yml`

```yaml
server:
  port: 8080 

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/meubanco
    username: root
    password: 123
  jpa:
    hibernate:
      ddl-auto: validate # Flyway/Liquibase assume o controle em prod
    show-sql: true
```

-----

## 💻 4. A Mão na Massa (Fluxo Completo)

### 🧩 Model (Entity) & 🔄 DTOs

```java
@Entity
@Table(name = "usuarios")
@Data
public class Usuario {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nome;
    private String email;
    private boolean ativo = true;
}

// O que o usuário envia
@Data
public class UsuarioCriarDTO {
    private String nome;
    private String email;
}

// O que nós devolvemos
@Data 
@AllArgsConstructor
public class UsuarioRespostaDTO {
    private Long id;
    private String nome;
    private String email;
}
```

### 🗄️ Repository & 🧠 Service

```java
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    boolean existsByEmail(String email);
}

@Service
public class UsuarioService {
    private final UsuarioRepository repository;

    // DIP: Injeção via construtor
    public UsuarioService(UsuarioRepository repository) {
        this.repository = repository;
    }

    public Usuario criar(UsuarioCriarDTO dto) {
        if (repository.existsByEmail(dto.getEmail())) {
            throw new RuntimeException("Email já existe");
        }
        
        Usuario user = new Usuario();
        user.setNome(dto.getNome());
        user.setEmail(dto.getEmail());
        
        return repository.save(user);
    }
}
```

### 🌐 Controller

```java
@RestController
@RequestMapping("/api/usuarios")
public class UsuarioController {
    private final UsuarioService service;

    public UsuarioController(UsuarioService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<UsuarioRespostaDTO> criar(@RequestBody UsuarioCriarDTO dto) {
        Usuario user = service.criar(dto);
        return ResponseEntity.ok(
            new UsuarioRespostaDTO(user.getId(), user.getNome(), user.getEmail())
        );
    }
}
```

-----

## 🛡️ 5. Tratamento de Erros e Boas Práticas

### 🚦 Padrões RESTful Inegociáveis

  * **URLs:** Sempre no plural e sem verbos (`GET /api/usuarios`).
  * **Aninhamento Curto:** Máximo de 2 níveis (`/api/pedidos/7/itens`).
  * **Status HTTP:** `200 OK`, `201 Created`, `204 No Content`, `400 Bad Request`, `404 Not Found`.

### 🚨 Exception Handler Global

```java
public record ErrorResponse(String timestamp, int status, String erro, Object detalhes, String path) {}

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> handle(Exception ex) {
        return ResponseEntity.badRequest().body(ex.getMessage());
    }
}
```

-----

## 🔒 6. Segurança (Spring Security & JWT)

A API adota o padrão **Stateless** com **JWT (JSON Web Tokens)**.

> [\!TIP]
> **Checklist de Segurança:**
>
>   * [x] Senhas criptografadas (BCrypt).
>   * [x] JWT Stateless (Enviado via header `Authorization: Bearer <token>`).
>   * [x] Variáveis de ambiente para segredos.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            );
        return http.build();
    }
}
```

-----

## 🚀 7. Escalabilidade e System Design

Quando escalar de **Monolito** para **Microsserviços**, as estratégias são:

  * **Comunicação Síncrona:** HTTP via **OpenFeign** com **Circuit Breaker** (Resilience4j) para evitar falhas em cascata.
  * **Comunicação Assíncrona:** Mensageria via **RabbitMQ** ou **Kafka** para desacoplar tarefas pesadas (ex: envio de e-mails, processamento de relatórios).
  * **Dados:** Implementação de **Redis** (`@Cacheable`) para alívio de banco de dados e estratégias de *Read Replicas* no PostgreSQL.

-----



<br>

> ⚡ *"Código limpo não é um luxo. É o mínimo."*

Desenvolvido com foco em engenharia por \<a href="https://www.google.com/search?q=https://github.com/Alef71"\>Álef Francisco Ribeiro Amaral


```
Gostaria que eu montasse um tutorial rápido de como você pode gravar um GIF da sua API rodando no Postman e adicionar ali no começo do README para deixar com ainda mais cara de "nível sênior"?
```
