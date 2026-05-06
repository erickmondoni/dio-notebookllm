Este repositório é o resultado de um projeto de Aprendizagem Ativa desenvolvido para o desafio da DIO, utilizando a Inteligência Artificial (NotebookLM) como instrutura acadêmica.

# 🎯 Contexto e Objetivos
Este repositório é o resultado de um projeto de **Aprendizagem Ativa** desenvolvido para o desafio da **DIO**, utilizando a Inteligência Artificial (**NotebookLM**). 

O objetivo era construir um NotebookLLM e definir como ele deveria se comportar como fonte de informação. Abaixo, o tema escolhido e um breve relato de como a IA se comportou.

## 🐍 Python for Network Engineers: Automação MikroTik & Infraestrutura como Código (IaC)

**Objetivos de Estudo:**
*   Dominar a **REST API do RouterOS v7** para extração de dados estruturados.
*   Implementar scripts de monitoramento em tempo real (ex: BGP, interfaces SFP).
*   Comparar métodos de automação: **CLI (Netmiko)** vs. **Nativo (API)**.
*   Explorar os fundamentos de **Infraestrutura como Código (IaC)** e fluxos **CI/CD**.

---

## 📚 Curadoria de Fontes
Abaixo estão os links reais utilizados para alimentar o NotebookLM, garantindo a curadoria entre fundamentos acadêmicos e prática de mercado:

### 🎥 Conteúdo em Vídeo (YouTube)
*   **[Harvard CS50P]** [Introduction to Programming with Python](https://www.youtube.com/watch?v=nLRL_NcnK-4) - Fundamentos e lógica de programação.
*   **[NIC.br - SemanaCap 5]** [Automatizando serviços de redes com Python: básico](https://www.youtube.com/watch?v=AHMQozWKRiY) - Foco em ISPs.
*   **[NIC.br - SemanaCap 8]** [Ferramentas de automação de redes para ISPs](https://www.youtube.com/watch?v=3Y7ohoDWMfY) - Orquestração e ferramentas de mercado.

### 📖 Documentação & Repositórios
*   **[MikroTik Help]** [REST API Official Documentation](https://help.mikrotik.com/docs/spaces/ROS/pages/47579162/REST+API) - Referência de Endpoints RouterOS v7.
*   **[GitHub]** [Netmiko Library](https://github.com/ktbyers/netmiko) - Automação via SSH para ambientes legados.
*   **[GitHub]** [RouterOS-API Python](https://github.com/socialwifi/RouterOS-API) - Implementação da API clássica.

---

## 🛠️ Engenharia de Prompts e "Cicatrizes"
Nesta seção, documento as estratégias de instrução da IA e os desafios técnicos superados (Troubleshooting).

### 🤖 Configuração do Mentor (Role Prompting)
Para elevar o nível das respostas, utilizei um **Prompt de Sistema** que define a "persona" do NotebookLM. Esta é uma técnica de reutilização que garante que a IA atue como um instrutor sênior:

> **Diretriz de Configuração:**
> "Atue como um Professor e Mentor em Automação de Redes. Não forneça apenas o código; explique o conceito (ex: métodos HTTP), compare métodos CLI (Netmiko) vs REST API (v7) e foque em boas práticas de ISP (Idempotência, Segurança e Performance)."

### Caso Real: Monitoramento de Vizinhos BGP
*   **Pergunta Estratégica:** *"Como criar um script Python que monitore o status de um vizinho BGP e envie um log se ele não estiver 'established'?"*
*   **Dificuldade (Cicatriz):** A IA inicialmente sugeriu o uso de Netmiko (SSH). Para um ISP com centenas de vizinhos, o *parsing* de texto via SSH é lento e propenso a falhas de processamento.
*   **Ajuste de Rota:** Utilizando a persona de "Mentor", forcei a comparação com a **REST API**. Identificamos que o retorno em **JSON** permite o uso de filtros nativos (Query Parameters) diretamente na URL, o que economiza memória do roteador e reduz drasticamente a latência da automação.

---

## 📖 Miniguia de Estudo (Entrega Final)

### 1. Resumos Estruturados
A automação de MikroTik na versão 7 marca a mudança para o paradigma "API-First":
*   **REST API:** Comunicação via HTTPS onde os dados já chegam como dicionários Python.
*   **Performance:** Filtros aplicados no Endpoint (lado do servidor) evitam o tráfego de dados desnecessários.
*   **Resiliência:** O uso de blocos `try/except` é obrigatório em ambientes ISP para tratar timeouts e falhas de rota de gerência.

### 2. Glossário de Conceitos
*   **JSON (JavaScript Object Notation):** Formato padrão de troca de dados lido nativamente pelo Python.
*   **Endpoint:** A URL de acesso a um recurso (ex: `/rest/routing/bgp/session`).
*   **Idempotência:** Propriedade de um script que pode ser executado várias vezes sem alterar o estado final se a configuração já estiver correta.
*   **Query Parameter:** Filtros adicionados à URL (ex: `?.query=state!="established"`) para refinar a busca.

### 3. Biblioteca de Prompts Reutilizáveis
Prompts configurados para revisões futuras no NotebookLM:
*   > *"Com base nas fontes, gere um script que busque sessões BGP filtradas pelo comentário 'Link_Principal'."*
*   > *"Explique como o conceito de Programação Assíncrona (asyncio) pode ser aplicado para consultar múltiplos roteadores simultaneamente via REST API."*
*   > *"Crie um checklist de segurança para expor a API do MikroTik apenas em uma VLAN de gerência isolada."*

---

## 🚀 Exemplo Prático de Implementação
Snippet de código recomendado para monitoramento performático via REST API:

```python
import requests

def check_bgp_status(ip, user, password):
    # Uso de Query Parameter para filtrar apenas vizinhos que NÃO estão established
    url = f"https://{ip}/rest/routing/bgp/session?.query=state!=\"established\""
    
    try:
        # Em produção, utilize certificados válidos em vez de verify=False
        response = requests.get(url, auth=(user, password), verify=False, timeout=5)
        # O roteador já retorna apenas os vizinhos com problema em formato JSON
        return response.json()
    except requests.exceptions.RequestException as e:
        return f"Erro de conexão com o roteador: {e}"

if __name__ == "__main__":
    # Teste de execução
    resultado = check_bgp_status("192.168.88.1", "admin", "password_secreta")
    print(resultado)
