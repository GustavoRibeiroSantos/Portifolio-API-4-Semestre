# Portifolio-API-4-Semestre

### Em 2021-1 JETSOFT - **[Gitlab do projeto](https://gitlab.com/gurst6/projeto-integrador-pythaon)**
O projeto tinha como problema a necessidade de performance em um gerenciador de currículos que contém diversos candidatos e vagas, além de ter uma inteligência que ajuda no cruzamento de vagas e currículos. Diversos novos candidatos e vagas eram inseridos todos os dias e era necessário que esses postos fossem preenchidos das maneiras mais eficiente possível. A inteligência era requisitada nesse momento, onde os requisitos para as vagas, como:
- Salário; 
- Qualificações; 
- Experiência profissional;
- VT0 (onde a distância percorrida pelo trabalhador até o trabalho não pode ser mais do que 1km).

Com isso, deveriam ser analisados de forma com que os possíveis candidatos estivessem o mais próximo possível dos requisitos demandados para a vaga. A solução proposta foi uma api que gerenciasse os currículos e vagas, utilizando de ferramentas do MongoDB para otimizar buscas e cruzamentos de postos de trabalho, além das queries geoespaciais para buscas por VT0.


### Tecnologias Utilizadas
* Python 3.8 - O python foi a liguagem principal para a programação da API.
* FrameWork Django 3 - Foi framework de projeto usado para a criação da aplicação.
* MongoDB - Banco de Dados usado para armazenar as informações de candidatos, vagas e etc.
    * **Pymongo** - interação com nosso Banco de Dados;

# Contribuições individuais/pessoais
A minha contribuição para o desenvolvimento do projeto foi tanto no back-end qunanto no front-end da nossa aplicação! 

### Criação da rota Web-Server para cadastro e atualização de curriculos 
- Eu fui o responsável no time por criar essas duas rotas das inserção de dados no nosso banco atraves das rotas Web Servers.

```
@csrf_exempt
   def cadastrarCurriculo(request):
      client = Vaga.createConnection()         
      db = client["Finder"]
      col = db["Inscrito"]

      result = col.insert_one(json.loads(request.body))
         
         # result
      return JsonResponse({"message":"Curriculo inserido com sucesso."}, status=200)

   @csrf_exempt
   def atualizarCurriculo(request, pk):
      client = Vaga.createConnection()
      db = client["Finder"]
      col = db["Inscrito"]

      result = col.update_one({"InscritoIdExterno" : pk}, json.loads(request.body))
         # result
      return JsonResponse({"message":"Curriculo atualizado com sucesso."}, status=200)
```
### Criação da rota Web-Server para atualizar a lista de curriculos que se aplicam para uma vaga
- Eu fui o responsável no time por criar essa rotas das inserção de dados no nosso banco atraves das rotas Web Servers, onde todas as vagas tinham uma lista com os curriculos que se encaixavam para ela, assim a busca no banco era bem mais rápida.

```
def UpdateListCurriculos(IdCol, pk):
      client = Vaga.createConnection()
      db = client["Finder"]
      col = db["vagas"]

      query = { "$set": { "IncritoIdSelecionado": IdCol }}

      result = col.update_one({"VagaIdExterno" : pk}, query)
      # result
      return JsonResponse({"message":"Lista de curriculos selecionados atualizada com sucesso."}, status=200)

```
### Criação da rota Web-Server para fazer uma busca das vagas no nosso banco
- Eu fui o responsável no time por criar essa rota e mostrar as vagas no banco.

```
@csrf_exempt
   def buscarPorVaga(VagaID):
      # Inicia conexão com o banco
      client = Vaga.createConnection()

      mydb = client["Finder"]
      curriculos = mydb["Inscrito"]
      vagas = mydb["vagas"]

         # Recupera a vaga recebida por parâmetro
      vaga = vagas.find_one({"VagaIdExterno" : VagaID})

      if vaga:
         searchRequisitos = '|'.join([str(requisito['descricao']) for requisito in vaga['competencia']])

         query = {
               "$or" : [ 
                  # { "tipoContratoDesejadoInscrito" : { "$regex": vaga['tipoContratacaoPerfilVaga'] } },
                  { "perfilProfissionalTituloInscrito" : { "$regex":searchRequisitos } },
                  { "perfilProfissionalDescricaoInscrito" : { "$regex": searchRequisitos } },
                  { "experienciaProfissional.descricao": { "$regex": searchRequisitos } },
                  { "formacao.curso": { "$regex": searchRequisitos } },
                  { "competencia.descricao": { "$regex": searchRequisitos } } 
               ] 
         }

         result_curriculos = curriculos.find(query)
         IdCol = [str(result['_id']) for result in result_curriculos]
      
      return IdCol
```
### Implementação do Observer no back-end da API
- Eu fui o responsável no time por implementar o observer na nossa aplicação, para nos ajudar na parte de atualização dos candidatos que se encaixam para uma vaga.

```
class Observer:

	observers = []
	view_func = None

	# def __init__(self): #Construtor da classe.
	# 	self.observers = []
	# 	self.view_func = views.View()
        

	def registerObserver(IdVaga):
		views.View.CurriculosList(IdVaga)
		Observer.observers.append(IdVaga)
		print("Observer added!")

	def removeObserver(self, IDVaga):
		for x in self.observers:
			if(IDVaga == x):
				self.observers.remove(x)

	def notifyObserver(IdVaga):
		results = views.View.CurriculosList(IdVaga)
		Observer.UpdateObserver(IdVaga, results)
		print("Finished!")
 
	def UpdateObserver(IDVaga, results):
		views.View.UpdateListCurriculos(results, IDVaga)
		print("Função para add curriculo a collection")
		print(dumps(results))

```

# Aprendizados Efetivos
Com base nas rotinas desenvolvidas, pode-se ser absorvido conhecimento sobre alguns pontos no back-end:
- Melhorei muito meu conhecimento com django;
- Entendi melhor como utilizar o MongoDB;
- Aprendi o que é o Observe, para que ser e como aplicar/implementar em uma aplicação;
- Entendi como funciona o Pymongo e como utiliz-lo.
