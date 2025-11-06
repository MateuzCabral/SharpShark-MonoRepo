# 🦈 SharpShark — Sistema de Análise de Tráfego e Segurança de Rede


## 📘 Descrição Geral
O **SharpShark** é um sistema de análise de tráfego e segurança de rede projetado para automatizar a análise forense de arquivos de captura de pacotes (.pcapng).

O projeto nasceu da necessidade de **acelerar a triagem de incidentes** e **reduzir a dependência de análises manuais complexas**, típicas do uso de ferramentas como o Wireshark.

O sistema implementa uma **pipeline de processamento automatizada**, composta por módulos de ingestão, análise, detecção e visualização, tornando a identificação de padrões maliciosos e anomalias acessível mesmo para analistas menos experientes.

---

## 🎯 Objetivo do Projeto

### Objetivo Geral
Desenvolver uma plataforma modular e extensível para **análise automatizada de tráfego de rede (.pcapng)**, capaz de identificar, correlacionar e reportar atividades suspeitas e anomalias estatísticas por meio de um motor de detecção heurístico.

### Objetivos Específicos
- Processar arquivos `.pcapng` e reconstruir fluxos de comunicação TCP.
- Gerar estatísticas de IPs, portas e protocolos mais utilizados.
- Detectar automaticamente padrões de ataques como **Brute Force** e assinaturas customizadas (ex: SQLi, XSS).
- Armazenar resultados em banco de dados SQLite.
- Controlar o acesso ao sistema através de autenticação de usuários (JWT).
- Exibir alertas e relatórios interativos via dashboard web (frontend React).

---

## 🧩 Arquitetura da Solução
O **SharpShark** segue uma arquitetura de **microsserviços containerizada**, orquestrada pelo **Docker Compose**, composta por dois componentes principais:

### 🔹 Backend (Python + FastAPI)
Responsável pela ingestão, parsing, análise, autenticação e persistência dos dados.

- **Ingestão:** Monitora uma “Hot Folder” (definida pelo usuário) com o módulo **Watchdog** e processa automaticamente novos arquivos `.pcapng`.
- **Análise:** Utiliza **PyShark (TShark)** para ler os pacotes e um motor de regras customizado para detecção.
- **Armazenamento:** Utiliza **SQLAlchemy** e **SQLite** para armazenar análises, alertas, usuários e regras.
- **API:** Expõe endpoints RESTful com **FastAPI** para o frontend.
- **Segurança:** Gerencia usuários e autenticação via token **JWT**.

**Principais Módulos (Corrigido):**
- `services/analysis.py`: Coração da análise. Faz o parsing dos pacotes, reconstrói streams e aplica o motor de regras.
- `services/ingestor.py`: Gerencia o Watchdog, monitora a "Hot Folder" e dispara novas análises.
- `services/auth.py`: Cuida da lógica de login e geração de token.
- `app/cli.py`: Script de linha de comando para criar o primeiro administrador.

---

### 🔹 Frontend (React + TypeScript + Nginx)
Interface web moderna e responsiva que atua como o client do sistema, servida por um container **Nginx**.

- **Interatividade:** Consome a API REST do backend para exibir e gerenciar dados.
- **Visualização:** Apresenta estatísticas, alertas e status com **Recharts** e componentes **shadcn/ui**.
- **Roteamento:** O Nginx atua como reverse proxy, direcionando chamadas de API (ex: `/api/files`) para o container backend e servindo o React para todas as outras requisições.

**Principais Componentes (Corrigido):**
- `pages/Dashboard.tsx`: Página principal com abas que agregam todos os módulos (Análises, Arquivos, Usuários, Regras, etc.).
- `pages/AnalysisDetails.tsx`: Visão detalhada de uma análise, mostrando estatísticas, IPs e alertas.
- `components/dashboard/UploadArea.tsx`: Componente para upload manual de arquivos `.pcapng`.
- `components/dashboard/UsersManagement.tsx`: CRUD completo para gerenciamento de usuários.
- `components/dashboard/CustomRules.tsx`: CRUD completo para gerenciamento de regras de detecção.

---

## ⚡ Como Executar o Projeto (Corrigido)
O projeto é **100% containerizado com Docker**.  
A instalação é universal e não requer Python, npm ou venv instalados na máquina host.

### 🔧 Pré-requisitos:
- **Git**
- **Docker Desktop** (ou Docker + Docker Compose)

### 1. Clone o Repositório
```bash
git clone https://github.com/MateuzCabral/SharpShark-Monorepo.git
cd SharpShark-Monorepo
```

### 2. Suba os Containers
Este é o comando principal. Ele irá:
- Construir a imagem do backend, instalando TShark e todas as dependências Python.
- Construir a imagem do frontend, compilando o React e configurando o Nginx.
- Iniciar os containers.

O script de inicialização do backend criará automaticamente o `.env` (se não existir), gerará uma `SECRET_KEY` segura e inicializará o banco de dados.

```bash
docker-compose up -d --build
```

*(Aguarde o build terminar. O primeiro pode demorar alguns minutos.)*

### 3. Crie seu Usuário Administrador
Com os containers rodando, execute este comando para criar seu primeiro usuário (necessário para login):

```bash
docker-compose exec backend python3 -m cli create-admin
```

Siga as instruções no terminal para definir seu nome e senha.

### 4. Acesse o Sistema
Pronto!  
Abra o navegador e acesse: **http://localhost**  
Use o usuário e senha de administrador criados no passo anterior.

---

## 📂 Estrutura do Repositório (Monorepo Corrigido)
```
SharpShark-Monorepo/
│
├── backend/            # Backend FastAPI (API, serviços, etc.)
│   ├── app/            # Código-fonte Python
│   │   ├── services/   # analysis.py, ingestor.py, auth.py, etc.
│   │   ├── api/        # Endpoints (rotas)
│   │   ├── db/         # Modelos SQLAlchemy (models.py)
│   │   ├── core/       # config.py, security.py
│   │   ├── cli.py      # Script para criar admin
│   │   └── main.py     # Ponto de entrada FastAPI
│   ├── Dockerfile      # Receita (Python + TShark)
│   └── entrypoint.sh   # Script de inicialização
│
├── frontend/           # Frontend React (UI)
│   ├── src/
│   │   ├── components/ # Componentes (UsersManagement.tsx, etc.)
│   │   ├── pages/      # Páginas (Dashboard.tsx)
│   │   └── api/        # Funções de fetch (auth.ts, files.ts)
│   ├── Dockerfile      # Receita (Node.js build -> Nginx)
│   └── nginx.conf      # Configuração do Reverse Proxy
│
├── captures/           # Pasta "Hot-Folder" (para arquivos .pcapng)
│   └── .gitkeep
│
├── docker-compose.yml  # Orquestrador principal
│
└── README.md
```

---

## 🔒 Requisitos Funcionais Principais

- **RF01:** Permitir upload manual de arquivos `.pcapng`.  
- **RF02:** Extrair IPs, portas, protocolos e payloads.  
- **RF03:** Reconstruir sessões de comunicação.  
- **RF04:** Gerar estatísticas de IPs, portas e protocolos.  
- **RF05:** Armazenar análises e alertas em SQLite.  
- **RF06:** Expor endpoints RESTful.  
- **RF07:** Exibir dashboards interativos.  
- **RF08:** Detalhar análises e alertas.  
- **RF09:** Indicar status das análises.  
- **RF10:** Autenticação de usuários via JWT.

---

## 🧠 Requisitos Não Funcionais

- **RNF01 (Performance):** Processar 100MB em até 5 minutos.  
- **RNF02 (Robustez):** Tolerar falhas de parsing.  
- **RNF03 (Usabilidade):** Interface intuitiva.  
- **RNF04 (Extensibilidade):** Inclusão de novas regras sem refatoração.  
- **RNF05 (Segurança):** Validação de arquivos de entrada.  
- **RNF06 (Documentação):** Manual de uso e API Swagger.

---

## 🧩 Critérios de Sucesso

✅ Processar arquivos de 100MB em menos de 5 minutos.  
✅ Detectar >90% de ameaças baseado nas regras definidas pelo usuário.  
✅ Usuário básico consegue interpretar relatórios.  
✅ Adicionar novas regras de detecção sem refatoração estrutural.  

---

## 🧰 Ferramentas Correlatas e Diferenciação

- **NetworkMiner:** Extração de artefatos e GUI intuitiva.  
  *Limitação:* Desktop, foco reduzido em dashboards.  

- **CapAnalysis:** Web-based com estatísticas de fluxos.  
  *Limitação:* Projeto pouco atualizado.  

- **🦈 SharpShark:** Combina backend automatizado (Python/FastAPI), interface moderna (React) e motor de detecção configurável, atuando como **primeira linha de defesa cibernética integrada**.


## 📚 Referências
- CapAnalysis  
- FastAPI  
- NetworkMiner  
- Python  
- React  
- Scapy  
- SQLAlchemy  
- Wireshark  