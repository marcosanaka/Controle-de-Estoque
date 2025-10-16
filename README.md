# Sistema — Controle de Estoque

Aplicação web simples (PHP + MySQL) para controle de produtos, posições em estoque, movimentações e clientes — com auditoria de movimentações, filtros, exportação CSV e gráficos de saída por mês.

## ✨ Principais recursos

- Autenticação com perfis (user/admin) e gestão de usuários (Admin)
- Cadastros de Produtos e Clientes com filtros de busca
- Estoque por posição (produto + local + lote + validade)
- Movimentar (entrada/saída/ajuste) com trilha de auditoria (estoque_mov_log)
- Histórico de Movimentações com filtros e exportação CSV
- Dashboard com KPIs de 30 dias e gráfico de saídas por mês (ano corrente)
- Paginação (10 itens/página) em todas as listas, preservando filtros
- Bloqueio de alteração de quantidade na edição de Estoque (alterações apenas via “Movimentar”)
- Favicon e branding KJM no cabeçalho e rodapé

## 🧱 Stack

- PHP 8.x (mysqli)
- MySQL/MariaDB
- Bootstrap 5.3
- Chart.js 4 (cópia local em `assets/js/chart.umd.min.js`)

## 📁 Estrutura do projeto

```
/var/www/html/kjm
├── index.php              # redireciona para login
├── login.php              # autenticação
├── dashboard.php          # app principal (rotas, views, migrações simples)
├── config/
│   └── db.php             # conexão com o banco
└── assets/
    ├── css/style.css
    ├── img/ (favicon, logos)
    └── js/
        ├── chart.umd.min.js
        └── dashboard_chart.js
```

## 🗄️ Banco de dados

O arquivo `dashboard.php` contém migrações idempotentes básicas para compatibilidade com bancos legados:

- Tabela `clientes` (e colunas complementares)
- Tabela de auditoria `estoque_mov_log` (log de todas as movimentações)
- Ajustes na tabela legada `movimentacoes` (quando presente)
- Módulo Admin utiliza tabela `users` (criada/ajustada quando necessário)

Campos de destaque em `estoque_mov_log`:
- id_estoque, id_produto, id_cliente (nullable)
- tipo (entrada/saida/ajuste), quantidade (int)
- valor_unitario, valor_total (nullable)
- user_id, user_name, data_mov

Observações:
- Ao criar uma nova posição de estoque, é registrado automaticamente um log de “entrada”.
- Na edição de estoque, a quantidade é bloqueada. Ajustes devem ser feitos via “Movimentar”.

## ⚙️ Configuração

1) Requisitos
- PHP 8.x com extensão mysqli
- MySQL/MariaDB acessível
- Servidor web (Apache/Nginx) ou servidor embutido do PHP

2) Configurar conexão
Edite `config/db.php` para seus dados de conexão. Exemplo:

```php
$mysqli = new mysqli('localhost', 'usuario', 'senha', 'estoque_db');
$mysqli->set_charset('utf8mb4');
```

3) Inicialização do banco
- Crie o schema (database) conforme especificado em `config/db.php`.
- A primeira execução aplicará as migrações idempotentes em tempo de execução.

## ▶️ Execução

- Apache/Nginx: aponte o DocumentRoot para a pasta do projeto.
- PHP embutido (para desenvolvimento):

```bash
php -S 0.0.0.0:8080 -t /var/www/html/kjm
```

Acesse http://localhost:8080/login.php

## 🔐 Acesso e perfis

- Login via `login.php`.
- Módulo Admin (menu "Admin") disponível apenas para usuários com `role=admin`.
- Admin pode criar/editar usuários e definir `role` (user/admin), além de ativar/desativar.

## 🧭 Módulos e rotas

- Produtos (`?m=produtos`)
  - Listar, filtrar, visualizar, criar/editar, excluir
  - Exibe estoque agregado por produto
- Clientes (`?m=clientes`)
  - Listar, filtrar, visualizar, criar/editar, excluir
  - Detalhes do cliente incluem “Movimentações do cliente” (mais recentes primeiro), com totalizador e paginação
- Estoque (`?m=estoque`)
  - Listagem por posição (produto, lote, validade, local, quantidade)
  - Nova posição / Editar posição (quantidade bloqueada em edição)
  - Movimentar (entrada/saída/ajuste) com validações (ex.: cliente obrigatório em saída)
  - Histórico (`?m=estoque&a=history`) com filtros, paginação e exportação CSV
- Admin > Usuários (`?m=admin`)
  - CRUD de usuários, troca de senha, filtros por username/nome

## 📊 Dashboard

- KPIs de últimos 30 dias (total de movimentações, entradas/saídas e valores)
- Gráfico “Saídas por mês (ano atual)” com:
  - Qtd de saídas (COUNT de registros)
  - Valor das saídas (SUM com fallback para `valor_total` ou `quantidade * valor_unitario`)
- Chart.js carregado localmente para maior resiliência

## ⬇️ Exportação CSV

- Em Histórico, clique em “Exportar CSV”.
- Exportação é tratada antes de qualquer HTML (evita misturar CSV com markup).
- Arquivo em UTF-8 com BOM; delimitador `;` para melhor compatibilidade com Excel.

## 🔎 Paginação e filtros

- Todas as listagens utilizam 10 itens por página.
- A paginação preserva automaticamente os filtros aplicados.

## 🧰 Desenvolvimento

- Código principal em `dashboard.php` (roteamento por querystring: `m`/`a`/`id`).
- Helpers diversos para HTML/segurança (ex.: `h()`), opções de selects e paginação.
- Front-end com Bootstrap 5.3 e tema simples em `assets/css/style.css`.
- Gráficos em `assets/js/dashboard_chart.js` lendo dados via JSON embutido na página.

## 🔒 Segurança

- Senhas de usuários com `password_hash`.
- Sessões controlam acesso; menu Admin protegido por `is_admin()`.
- Importante: não versionar credenciais sensíveis. Ajuste `config/db.php` com variáveis de ambiente ou um arquivo local fora do VCS, quando possível.

## 🪪 Branding & Rodapé

- Cabeçalho com logo e texto “Controle de Estoque”.
- Rodapé: `© 2025  Controle de Estoque- Desenvolvido por Marcos Roberto`.

## 📜 Licença

Defina a licença do projeto conforme sua necessidade (ex.: Proprietária). Adicione um arquivo `LICENSE` se aplicável.
