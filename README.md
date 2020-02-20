#nesse programa, eu pensei em como funciona um back-end de um sistema de login.
#e depois fui apreendendo e colocando em prática com mais complexidade... ai dividi o programa em funções e usei a biblioteca matplotlib para gerar gráficos
#NESSE PROJETO APRENDI
#.CONSULTAR O MYSQL
#.FAZER UM SISTEMA DE CADASTRO E LOGIN PARA PESSOA
#USAR UMA BIBLIOTECA PARA GRÁFICOS 
#EXCLUIR DADOS DO MYSQL
#ETC...








import matplotlib.pyplot as plt
import pymysql.cursors

#CONEXÃO COM BANCO DE DADOS
conexao = pymysql.connect(
    host='localhost',
    user='root',
    passwd='',
    database='erp',
    charset='utf8mb4',
    cursorclass= pymysql.cursors.DictCursor

    
)

autentico = False

def logarcadastrar():
#NESSA PARTE DA FUNÇÃO LOGAR CADASTRAR DEPEDENDO DA OPÇÃO DO  USUÁRIO O SISTEMA VERIFICA SE BATE O DADOS CONSULTANDO O MYSQL



    usuarioexistente=0
    autenticado= False
    usuariomaster= False

    if decisao == 1:
        nome=input('digite o nome\n')
        senha= input('digite a senha\n')

        #percorrer o banco de daddos para conferir o nome e a senha
        for linha in resultado:
            if nome == linha['nome'] and senha == linha['senha']:
                if linha['nivel'] ==1:
                    usuariomaster= False
                elif linha['nivel']==2:
                    usuariomaster= True
                autenticado = True
                print('logado')
                break
            else:
                autenticado= False
        if not autenticado:
            print('nome ou senha errado')


    elif decisao == 2:
         nome= input("digite o nome\n")
         senha= input('digite a senha\n')
         for linha in resultado:
            if nome == linha['nome'] and senha == linha['senha']:
                usuarioexistente = 1

         if usuarioexistente==1:
            print("nome e senha já caadastrado")
        
         elif usuarioexistente == 0:
            try:
                with conexao.cursor() as cursor:
                    cursor.execute('insert into cadastros(nome,senha,nivel) values (%s,%s,%s)', (nome,senha,1))
                    conexao.commit()
                print('usuario cadastrado com sucesso')

            except:
                print('erro ao inserir no banco de dados, tente novamente')

    return autenticado, usuariomaster
def cadastarprodutos():
#AQUI JÁ É A PARTE DE CADASTRAR UM PRODUTO E O SISTEMA INSERI AO BANCO DE DADOS
    nome=input('digite o nome do produto')
    ingrediente=input('digite os ingredientes do produto')
    grupo=input('digite a que grupo pertence')
    preco=(input('digite o preço'))

    try:
        with conexao.cursor() as cursor:
            cursor.execute('insert into produtos(nome, ingredientes,grupo,preco) values (%s,%s,%s,%s)', (nome,ingrediente,grupo,preco))
            conexao.commit()
            print('produto cadastrado')
    except:
        print('erro ao inserir os produtos no banco de dados')
def listarprodutos():
# AQUI ELE JÁ LISTA OS PRODUTOS DENTRO DE UMA LISTA, ELE CONSULTA O BANCO DE DADOS E SE TIVER PRODUTOS, ELE MANDA PARA UMA LISTA PARA MOSTRAR NA TELA DE UMA FORMA MAIS ORGANIZADA
    produtos = []

    try:
        with conexao.cursor() as cursor:
            cursor.execute('select * from produtos')
            produtoscadastrados = cursor.fetchall()
            print(produtoscadastrados)

    except:
        print('erro ao conectar ao banco de dados')

    #percorrer os produtos e listar de uma forma organizada
    for i in produtoscadastrados:
        produtos.append(i)

    if len(produtos) !=0:
        for i in range(0, len(produtos)):
            print(produtos[i])
    else:
        print('nenhum produto cadstrado')
def excluirproduto():
#NESSA PARTE O PROGRAMA EXCLUI O PRODUTO DE ACORDO COM SEU ID
    excluir= int(input('digite o nome do id do produto'))

    try:
        with conexao.cursor() as cursor:
            cursor.execute('delete from produtos where id = {}'.format(excluir))    
            print('produto exluido com sucesso')
    except:
        print('erro ao excluir o produto')
def listarpedidos():
#AQUI PENSAMENTO EM UMA LANCHONETE, ELE MOSTRA OS PEDIDOS E ATUALIZA SER O PEDIDO JA FOI ENTREGUE
    pedidos = []
    decisao =0

    while decisao !=2:
        pedidos.clear()

        try:
            with conexao.cursor() as cursor:
                cursor.execute('select * from pedidos')
                listarpedidos = cursor.fetchall()

        except:
            print('erro ao conectar ao bd')


        for i in listarpedidos:
            pedidos.append(i)

        if len(pedidos) !=0:
            for i in range(0, len(pedidos)):
                print(pedidos[i])
        else:
            print('nenhum pedido foi feito')

        decisao = int(input('digite 1 para dar produto com entregue e 2 para voltar'))
        if decisao ==1:
           iddeletar = int(input('digite o id do produto entregue'))
           try:
               with conexao.cursor() as cursor:
                   cursor.execute('delete  from pedidos where id = {} '.format(iddeletar))
                   print('produto dado como entregue')
                
           except:
                print('erro ao apagar o pedido')
def gerandoestatiticas():
    nomeProdutos = []
    nomeProdutos.clear()

    try:
        with conexao.cursor() as cursor:
            cursor.execute('select * from produtos')
            produtos = cursor.fetchall()
    except:
        print('erro ao fazer consulta no banco de dados')

    try:
        with conexao.cursor() as cursor:
            cursor.execute('select * from estatisticaVendido')
            vendido = cursor.fetchall()
    except:
        print('erro ao fazer a consulta no banco de dados')


    estado = int(input('digite 0 para sair, 1 para pesquisar por nome e 2 para pesquisar por grupo'))

    if estado == 1:
        decisao3 = int(input('digite 1 para pesquisar por dinheiro e 2 por quantidade unitaria'))
        if decisao3 == 1:

            for i in produtos:
                nomeProdutos.append(i['nome'])

            valores = []
            valores.clear()

            for h in range(0, len(nomeProdutos)):
                somaValor = -1
                for i in vendido:
                    if i['nome'] == nomeProdutos[h]:
                        somaValor += i['preco']
                if somaValor == -1:
                    valores.append(0)
                elif somaValor > 0:
                    valores.append(somaValor+1)

            plt.plot(nomeProdutos, valores)
            plt.ylabel('quantidade vendida em reais')
            plt.xlabel('produtos')
            plt.show()

        if decisao3 == 2:
            grupounico = []
            grupounico.clear()

            try:
                with conexao.cursor() as cursor:
                    cursor.execute('select * from produtos')
                    grupo= cursor.fetchall()

            except:
                print('erro ao cadastrar ao banco de dados 1')

            try:
                with conexao.cursor() as cursor:
                    cursor.execute('select  * from  estatisticaVendido')
                    vendidogrupo = cursor.fetchall()
            except:
                print('erro ao consultar a bd 2')

            for i in grupo:
                grupounico.append(i['nome'])
#NESSA FUNÇÃO ELE GERA ESTATITICA DE ACORDO COM AS VENDAS DOS PRODUTOS, SEJA DE VALOR MONÉTARIO OU DE UNITÁRIO
#aqui é para que o produto nao se repita na tela...
            grupounico=sorted(set(grupounico))

            quantfinal = []
            quantfinal.clear()

            for h in range(0, len(grupounico)):
                quantunitaria = 0
                for i in vendidogrupo:
                     if grupounico[h] == i['nome']:
                        quantunitaria +=1
                quantfinal.append(quantunitaria)

            plt.plot(grupounico, quantfinal)
            plt.ylabel('quantidade vendida unitaria')
            plt.xlabel('produtos')
            plt.show()
    
    elif estado  ==2:
        decisao3 = int(input('digite 1 para pesquisar por dinheiro e 2 por quantidade unitaria'))
        if decisao3 == 1:

            for i in produtos:
                nomeProdutos.append(i['grupo'])

            valores = []
            valores.clear()

            for h in range(0, len(nomeProdutos)):
                somaValor = -1
                for i in vendido:
                    if i['grupo'] == nomeProdutos[h]:
                        somaValor += i['preco']
                if somaValor == -1:
                    valores.append(0)
                elif somaValor > 0:
                    valores.append(somaValor + 1)

            plt.plot(nomeProdutos, valores)
            plt.ylabel('quantidade vendida em reais')
            plt.xlabel('produtos')
            plt.show()




#corpo da estrutura aonde fica repedindo a pergunta:
while not autentico:
    decisao = int(input('digite 1 para logar e 2 para cadastrar\n'))

    try:
        with conexao.cursor() as cursor:
            cursor.execute('select * from cadastros')
            resultado= cursor.fetchall()
            
    except:
        print("erro ao conectar ao bd")
    autentico,usuariosupremo = logarcadastrar()


if autentico:
    print('autenticado')
    decisaousuario = 1

    if usuariosupremo == True:
    
        while decisaousuario != 0:
             decisaousuario=int(input('digite 0 para sair e 1 para cadastrar o produto e 2 para  listar produtos e 3 para listra  pedidos e 4 para gerar estátitica'))

             if decisaousuario ==1:
                cadastarprodutos()
             if decisaousuario ==2:
                 listarprodutos()
                 delete= int(input('digite 1 para sair e 2 para exluir o produto novamente'))
                 if delete == 2:
                    excluirproduto()
             elif decisaousuario == 3:
                  listarpedidos()

             elif decisaousuario ==4:
                gerandoestatiticas()
                
