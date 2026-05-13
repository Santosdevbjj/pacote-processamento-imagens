# Pacote de processamento de imagena: Biblioteca Python para Processamento de Imagens Publicada no PyPI

![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat&logo=python&logoColor=white)
![Pillow](https://img.shields.io/badge/Pillow-10.0+-3776AB?style=flat&logo=python&logoColor=white)
![Pytest](https://img.shields.io/badge/Pytest-7.0+-0A9EDC?style=flat&logo=pytest&logoColor=white)
![PyPI](https://img.shields.io/badge/PyPI-Publicável-006DAD?style=flat&logo=pypi&logoColor=white)
![MIT License](https://img.shields.io/badge/Licença-MIT-22C55E?style=flat)

> Pacote Python modular para processamento de imagens — redimensionamento, conversão para escala de cinza, aplicação de filtros visuais e listagem em lote — estruturado com separação de responsabilidades, suíte de testes automatizados e pipeline completo de empacotamento e publicação no PyPI.

---

## 1. Problema de Negócio

Pipelines de visão computacional, automação de mídias e sistemas de pré-processamento de imagens repetem as mesmas operações fundamentais em projetos diferentes: redimensionar, converter para escala de cinza, aplicar filtros de detecção de borda ou suavização, listar arquivos em lote.

O problema não é a complexidade dessas operações — o Pillow resolve cada uma delas em poucas linhas. O problema é **reuso e distribuição**: sem empacotamento adequado, o mesmo código é copiado de projeto em projeto, gerando inconsistências de versão, ausência de testes e dependência implícita de configurações locais.

O desafio central deste projeto é demonstrar o ciclo completo de **desenvolvimento → teste → empacotamento → publicação** de uma biblioteca Python reutilizável: do código funcional ao pacote instalável via `pip install`, disponível publicamente no PyPI.

---

## 2. Contexto

O projeto foi desenvolvido no **Bootcamp Suzano — Python Developer #2**, com objetivo de ir além da implementação das funções e construir o artefato completo que qualquer desenvolvedor Python precisa dominar: um pacote distribuível.

A escolha de processamento de imagens como domínio não é arbitrária — é uma área com demanda crescente em pipelines de Machine Learning (pré-processamento de datasets), sistemas de automação de conteúdo visual e aplicações de visão computacional. As operações implementadas — redimensionamento, conversão para cinza, aplicação de filtros, listagem em lote — são exatamente as etapas de normalização que precedem modelos de classificação e detecção de objetos.

A arquitetura do pacote segue separação de responsabilidades em quatro módulos (`core`, `filters`, `utils`, `demo`), convenções de empacotamento Python (`setup.py`, `MANIFEST.in`, `find_packages`) e suíte de testes com `pytest` usando fixtures de arquivos temporários — estrutura diretamente replicável em qualquer biblioteca Python de produção.

---

## 3. Premissas

- As operações de processamento retornam o objeto `Image` do Pillow além de salvar o arquivo — permitindo encadeamento de transformações sem I/O intermediário desnecessário.
- Filtros inválidos passados a `aplicar_filtro()` fazem fallback automático para `BLUR` ao invés de lançar exceção — decisão de robustez para pipelines automatizados onde o filtro pode ser configurado externamente.
- Os testes usam `tmp_path` (fixture nativa do pytest) para criar e destruir arquivos temporários — garantindo isolamento total entre casos de teste sem poluir o repositório.
- O parâmetro `salvar_como` é opcional em todas as funções de processamento: sem ele, a função opera em memória e retorna o objeto `Image` sem I/O — padrão adequado para pipelines onde o resultado é consumido diretamente por outro passo.
- A versão `0.1.0` segue versionamento semântico; qualquer quebra de interface pública incrementa o major.

---

## 4. Estratégia da Solução

### 4.1 Arquitetura Modular em Quatro Responsabilidades

```
src/pacote_process_imagem/
├── __init__.py     → Exposição pública da API e metadados do pacote
├── core.py         → Operações fundamentais: redimensionamento e conversão
├── filters.py      → Aplicação de filtros visuais via ImageFilter
├── utils.py        → Funções auxiliares: listagem de imagens em lote
└── demo.py         → Script de demonstração executável
tests/
├── test_core.py    → Testes de redimensionamento e conversão para cinza
├── test_filters.py → Testes de filtros válidos, inválidos e dimensões preservadas
└── test_utils.py   → Testes de listagem e filtragem por extensão
docs/
├── tutorial_instalacao.md    → Guia de instalação local
└── guia_publicacao_pypi.md   → Ciclo completo de publicação no PyPI
```

### 4.2 API Pública do Pacote

Três funções expostas diretamente via `__init__.py`:

```python
from pacote_process_imagem import redimensionar_imagem, converter_para_cinza, aplicar_filtro

# Redimensiona para 200x200 e salva
redimensionar_imagem("foto.jpg", 200, 200, "foto_menor.jpg")

# Converte para escala de cinza
converter_para_cinza("foto.jpg", "foto_pb.jpg")

# Aplica filtro: BLUR | CONTOUR | DETAIL | EDGE_ENHANCE
aplicar_filtro("foto.jpg", "CONTOUR", "foto_filtro.jpg")

# Lista imagens em um diretório (.jpg, .png, .jpeg)
from pacote_process_imagem.utils import listar_imagens
imagens = listar_imagens("./dataset/")
```

### 4.3 Suíte de Testes com Isolamento por Fixtures

| Arquivo de Teste | Casos Cobertos |
|---|---|
| `test_core.py` | Conversão para cinza com verificação de arquivo gerado |
| `test_filters.py` | BLUR, CONTOUR, filtro inválido (fallback), dimensões preservadas |
| `test_utils.py` | Listagem filtrando por extensão, retorno vazio em diretório sem imagens |

Todos os testes usam `tmp_path` — nenhum arquivo persiste após a execução da suíte.

### 4.4 Pipeline de Empacotamento e Publicação

```bash
# 1. Limpar builds anteriores
rm -rf build dist *.egg-info

# 2. Gerar distribuições (sdist + wheel)
python setup.py sdist bdist_wheel

# 3. Validar no TestPyPI
python -m twine upload --repository testpypi dist/*

# 4. Instalar do TestPyPI para validar
pip install --index-url https://test.pypi.org/simple/ pacote-process-imagem

# 5. Publicar oficialmente
python -m twine upload dist/*

# 6. Instalar a versão pública
pip install pacote-process-imagem
```

O `setup.py` configura `find_packages(where='src')` com `package_dir={'': 'src'}` — padrão correto para pacotes com código-fonte em subdiretório, que o `pip` e o `setuptools` resolvem automaticamente ao instalar.

---

## 5. Decisões Técnicas

### Por que Pillow ao invés de OpenCV para este pacote?

OpenCV oferece mais recursos para visão computacional avançada, mas carrega dependências pesadas (binários C++) que complicam distribuição via PyPI. Para um pacote de operações fundamentais de pré-processamento, Pillow é a escolha correta: instalação pura via `pip`, sem dependências de sistema, compatível com qualquer ambiente — incluindo containers mínimos e serverless. A migração para OpenCV faz sentido quando o pacote evoluir para detecção de objetos ou processamento de vídeo.

### Por que `setup.py` ao invés de `pyproject.toml`?

`pyproject.toml` com `build-backend = "setuptools.build_meta"` é o padrão moderno recomendado pela PEP 517/518. A escolha por `setup.py` foi pragmática para o contexto do bootcamp — mais documentado em tutoriais e com comportamento mais explícito para quem está aprendendo o ciclo de empacotamento pela primeira vez. Em produção, a migração para `pyproject.toml` é o próximo passo natural e está documentada nos próximos passos.

### Por que fallback silencioso para `BLUR` em filtros inválidos ao invés de exceção?

Em pipelines automatizados onde o nome do filtro vem de configuração externa (arquivo YAML, variável de ambiente, parâmetro de API), uma exceção por filtro inválido interrompe o processamento inteiro do lote. O fallback para `BLUR` garante que o pipeline continue e o resultado seja sempre um arquivo válido — o comportamento inesperado é logável, não fatal. Para casos onde a falha explícita é necessária, a assinatura pode receber `strict=True` como parâmetro opcional na próxima versão.

### Por que `MANIFEST.in` explícito?

Sem `MANIFEST.in`, o `setuptools` inclui apenas arquivos `.py` na distribuição `sdist`. Isso significa que `README.md`, `LICENSE`, testes e documentação seriam omitidos do tarball — quebrando a expectativa de quem baixa o pacote-fonte para inspecionar ou contribuir. O `MANIFEST.in` garante que a distribuição seja completa e auditável.

---

## 6. Insights

A implementação do ciclo completo de empacotamento revelou comportamentos não óbvios com impacto real:

**`find_packages(where='src')` requer `package_dir={'': 'src'}` simultâneo:** sem o segundo parâmetro, o setuptools encontra os pacotes mas não sabe onde buscá-los durante a instalação. O erro se manifesta apenas ao instalar o pacote gerado — não durante o desenvolvimento local — tornando-o difícil de diagnosticar sem executar o ciclo completo de publicação e instalação.

**`tmp_path` do pytest é mais robusto que `tempfile.mkdtemp()` manual:** a fixture gerencia criação e destruição automaticamente, inclusive em caso de falha no teste. Testes que criam arquivos com `tempfile` manual frequentemente deixam resíduos quando a asserção falha antes do cleanup — problema que `tmp_path` elimina por design.

**Retornar o objeto `Image` além de salvar o arquivo dobra a utilidade das funções:** uma função que só salva e retorna `None` força o chamador a reabrir o arquivo para continuar o processamento. O padrão `salvar_se_caminho_fornecido + retornar_objeto` permite tanto uso em pipeline em memória quanto persistência em disco com a mesma chamada — sem I/O desnecessário.

**O `MANIFEST.in` com `recursive-include src *` inclui `__pycache__/`:** em distribuições limpas, o correto é adicionar `global-exclude *.py[cod]` e `prune */\__pycache__` para evitar bytecode compilado na distribuição-fonte.

---

## 7. Resultados

- **Pacote Python instalável via `pip`** com API pública de três funções, metadados de versão e autor declarados em `__init__.py`, e ciclo completo de publicação documentado.

- **Quatro módulos com separação explícita de responsabilidades:** operações fundamentais (`core`), filtros visuais (`filters`), utilitários de I/O (`utils`) e demonstração executável (`demo`) — cada módulo testável e substituível independentemente.

- **Suíte de testes com 7 casos** cobrindo comportamento nominal, filtros inválidos com fallback, preservação de dimensões e listagem com filtragem de extensão — todos isolados por `tmp_path`, sem dependência de arquivos externos.

- **Pipeline de publicação documentado em dois ambientes:** TestPyPI para validação e PyPI oficial para distribuição — incluindo o ciclo completo de limpeza, geração de distribuições `sdist` e `wheel`, upload e verificação de instalação.

- **Estrutura diretamente replicável** para qualquer biblioteca Python: a separação `src/pacote/`, `tests/`, `docs/` com `setup.py`, `MANIFEST.in` e `requirements.txt` serve de template para novos pacotes sem retrabalho de configuração.

---

## 8. Tecnologias Utilizadas

| Tecnologia | Versão | Papel no Projeto |
|---|---|---|
| Python | 3.8+ | Linguagem principal |
| Pillow | 10.0+ | Processamento de imagens (resize, conversão, filtros) |
| Pytest | 7.0+ | Suíte de testes automatizados com fixtures |
| Setuptools | — | Empacotamento: `find_packages`, `bdist_wheel`, `sdist` |
| Twine | — | Upload seguro para TestPyPI e PyPI |
| Git / GitHub | — | Versionamento e hospedagem |

---

## 9. Como Executar

### Instalação via pip (após publicação no PyPI)

```bash
pip install pacote-process-imagem
```

### Instalação local para desenvolvimento

```bash
# Clone o repositório
git clone https://github.com/Santosdevbjj/pacote-processamento-imagens.git
cd pacote-processamento-imagens

# Instale as dependências
pip install -r requirements.txt

# Instale o pacote em modo editável (alterações no src/ refletem imediatamente)
pip install -e .
```

### Executar a demonstração

```bash
python -m src.pacote_process_imagem.demo
```

Gera três arquivos de saída: `saida_redimensionada.jpg`, `saida_cinza.jpg`, `saida_filtro.jpg`.

### Rodar a suíte de testes

```bash
pytest tests/ -v
```

### Uso em código

```python
from pacote_process_imagem import redimensionar_imagem, converter_para_cinza, aplicar_filtro
from pacote_process_imagem.utils import listar_imagens

# Processar imagem individual
redimensionar_imagem("foto.jpg", 640, 480, "foto_hd.jpg")
converter_para_cinza("foto.jpg", "foto_pb.jpg")
aplicar_filtro("foto.jpg", "EDGE_ENHANCE", "foto_bordas.jpg")

# Processar lote de imagens em um diretório
for nome_arquivo in listar_imagens("./dataset/"):
    converter_para_cinza(f"./dataset/{nome_arquivo}", f"./processado/{nome_arquivo}")
```

### Publicar no PyPI

Consulte o guia completo em [`docs/guia_publicacao_pypi.md`](docs/guia_publicacao_pypi.md).

---

## 10. Aprendizados

**O maior aprendizado técnico foi entender a diferença entre "funciona localmente" e "funciona instalado".** Durante o desenvolvimento, `from pacote_process_imagem import ...` funcionava porque o `src/` estava no path do Python via IDE. Ao instalar o pacote gerado em um ambiente limpo, os imports falhavam — o `package_dir={'': 'src'}` no `setup.py` estava ausente. Esse tipo de problema só aparece quando se executa o ciclo completo de `pip install .` em um virtualenv isolado, não durante o desenvolvimento.

**Sobre design de API para bibliotecas:** a decisão de retornar o objeto `Image` em todas as funções (além de salvar opcionalmente) surgiu ao tentar encadear operações — redimensionar e depois converter para cinza sem regravar o arquivo intermediário. Bibliotecas bem projetadas permitem uso em memória e em disco com a mesma interface; esse princípio guia o design de qualquer função que processe e produza dados.

**O que faria diferente hoje:** migraria o `setup.py` para `pyproject.toml` (PEP 517/518), adicionaria `pytest-cov` com relatório de cobertura no CI, incluiria `global-exclude *.py[cod]` no `MANIFEST.in` para evitar bytecode na distribuição-fonte e implementaria uma função `processar_lote()` em `utils.py` que aplica qualquer transformação a todos os arquivos de um diretório — reduzindo o boilerplate do exemplo de uso de lote acima.

---

## 11. Próximos Passos

- Migrar `setup.py` para `pyproject.toml` seguindo PEP 517/518 e `build-backend = "setuptools.build_meta"`
- Adicionar `pytest-cov` com threshold mínimo de cobertura no pipeline de CI via GitHub Actions
- Implementar `processar_lote(pasta, operacao, **kwargs)` em `utils.py` para aplicar qualquer transformação a um diretório inteiro
- Adicionar suporte a `SHARPEN` e `EMBOSS` no módulo `filters.py`, expandindo o catálogo de filtros disponíveis
- Implementar verificação de tipo e formato de arquivo na abertura de imagens, com exceção descritiva para formatos não suportados
- Criar workflow GitHub Actions para publicação automática no PyPI a cada tag de versão (`on: push: tags: v*`)

---

## Autor

**Sergio Santos** — Senior Data Engineer & Cloud Architect

[![Portfólio](https://img.shields.io/badge/Portfólio-Sérgio_Santos-111827?style=for-the-badge&logo=githubpages&logoColor=00eaff)](https://portfoliosantossergio.vercel.app)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Sérgio_Santos-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/santossergioluiz)
[![GitHub](https://img.shields.io/badge/GitHub-Santosdevbjj-24292f?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Santosdevbjj)
