
from abc import ABC, abstractmethod
from datetime import datetime
import textwrap


class RegistroOperacoes:
    def __init__(self):
        self._eventos = []

    @property
    def eventos(self):
        return self._eventos

    def adicionar(self, operacao):
        self._eventos.append({
            "tipo": operacao.__class__.__name__,
            "valor": operacao.valor,
            "data": datetime.now().strftime("%d/%m/%Y %H:%M:%S")
        })


class ClienteBanco:
    def __init__(self, endereco):
        self.endereco = endereco
        self.contas = []

    def executar_operacao(self, conta, operacao):
        operacao.registrar(conta)

    def adicionar_conta(self, conta):
        self.contas.append(conta)


class ClientePF(ClienteBanco):
    def __init__(self, nome, nascimento, cpf, endereco):
        super().__init__(endereco)
        self.nome = nome
        self.nascimento = nascimento
        self.cpf = cpf


class ContaBancaria:
    def __init__(self, numero, titular):
        self._saldo_atual = 0.0
        self._numero = numero
        self._agencia = "0001"
        self._titular = titular
        self._registro = RegistroOperacoes()

    @classmethod
    def criar(cls, titular, numero):
        return cls(numero, titular)

    @property
    def saldo(self):
        return self._saldo_atual

    @property
    def numero(self):
        return self._numero

    @property
    def agencia(self):
        return self._agencia

    @property
    def historico(self):
        return self._registro

    @property
    def titular(self):
        return self._titular

    def depositar(self, valor):
        if valor <= 0:
            print("Valor inválido.")
            return False

        self._saldo_atual += valor
        print("Depósito realizado com sucesso.")
        return True

    def sacar(self, valor):
        if valor <= 0:
            print("Valor inválido.")
            return False

        if valor > self._saldo_atual:
            print("Saldo insuficiente.")
            return False

        self._saldo_atual -= valor
        print("Saque realizado com sucesso.")
        return True


class ContaCorrentePF(ContaBancaria):
    def __init__(self, numero, titular, limite=500, limite_saques=3):
        super().__init__(numero, titular)
        self.limite = limite
        self.limite_saques = limite_saques

    def sacar(self, valor):
        quantidade_saques = len(
            [x for x in self.historico.eventos if x["tipo"] == "OperacaoSaque"]
        )

        if valor > self.limite:
            print("Limite por saque excedido.")
            return False

        if quantidade_saques >= self.limite_saques:
            print("Limite diário de saques atingido.")
            return False

        return super().sacar(valor)

    def __str__(self):
        return (
            f"Agência: {self.agencia}\n"
            f"Conta: {self.numero}\n"
            f"Titular: {self.titular.nome}"
        )


class Operacao(ABC):

    @property
    @abstractmethod
    def valor(self):
        pass

    @abstractmethod
    def registrar(self, conta):
        pass


class OperacaoDeposito(Operacao):
    def __init__(self, valor):
        self._valor = valor

    @property
    def valor(self):
        return self._valor

    def registrar(self, conta):
        if conta.depositar(self.valor):
            conta.historico.adicionar(self)


class OperacaoSaque(Operacao):
    def __init__(self, valor):
        self._valor = valor

    @property
    def valor(self):
        return self._valor

    def registrar(self, conta):
        if conta.sacar(self.valor):
            conta.historico.adicionar(self)


clientes = []
contas_ativas = []


def localizar_cliente(cpf):
    for cliente in clientes:
        if cliente.cpf == cpf:
            return cliente
    return None


def cadastrar_cliente():
    cpf = input("CPF: ")

    if localizar_cliente(cpf):
        print("Cliente já cadastrado.")
        return

    nome = input("Nome: ")
    nascimento = input("Data de nascimento: ")
    endereco = input("Endereço: ")

    clientes.append(
        ClientePF(nome, nascimento, cpf, endereco)
    )

    print("Cliente cadastrado.")


def abrir_conta():
    cpf = input("CPF do titular: ")
    cliente = localizar_cliente(cpf)

    if not cliente:
        print("Cliente não encontrado.")
        return

    numero = len(contas_ativas) + 1

    conta = ContaCorrentePF(numero, cliente)

    cliente.adicionar_conta(conta)
    contas_ativas.append(conta)

    print("Conta criada com sucesso.")


def listar_contas():
    if not contas_ativas:
        print("Nenhuma conta cadastrada.")
        return

    for conta in contas_ativas:
        print("-" * 40)
        print(conta)


def selecionar_conta():
    numero = int(input("Número da conta: "))

    for conta in contas_ativas:
        if conta.numero == numero:
            return conta

    return None


def realizar_deposito():
    conta = selecionar_conta()

    if not conta:
        print("Conta não encontrada.")
        return

    valor = float(input("Valor do depósito: "))

    conta.titular.executar_operacao(
        conta,
        OperacaoDeposito(valor)
    )


def realizar_saque():
    conta = selecionar_conta()

    if not conta:
        print("Conta não encontrada.")
        return

    valor = float(input("Valor do saque: "))

    conta.titular.executar_operacao(
        conta,
        OperacaoSaque(valor)
    )


def mostrar_extrato():
    conta = selecionar_conta()

    if not conta:
        print("Conta não encontrada.")
        return

    print("\\n===== EXTRATO =====")

    if not conta.historico.eventos:
        print("Sem movimentações.")
    else:
        for evento in conta.historico.eventos:
            print(
                f"{evento['tipo']} | "
                f"R$ {evento['valor']:.2f} | "
                f"{evento['data']}"
            )

    print(f"Saldo: R$ {conta.saldo:.2f}")


def menu():
    texto = """
    [1] Cadastrar cliente
    [2] Abrir conta
    [3] Depositar
    [4] Sacar
    [5] Extrato
    [6] Listar contas
    [0] Sair
    """
    return input(textwrap.dedent(texto) + "\\nEscolha: ")


def iniciar():
    acoes = {
        "1": cadastrar_cliente,
        "2": abrir_conta,
        "3": realizar_deposito,
        "4": realizar_saque,
        "5": mostrar_extrato,
        "6": listar_contas,
    }

    while True:
        opcao = menu()

        if opcao == "0":
            break

        funcao = acoes.get(opcao)

        if funcao:
            funcao()
        else:
            print("Opção inválida.")


if __name__ == "__main__":
    iniciar()
