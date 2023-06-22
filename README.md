# bloc_control_client

A new Flutter project.

## 📚 Getting Started

Neste projeto veremos como utilizar a biblioteca Bloc para realizar  requisições HTTP e gerenciar o estado de nossa aplicação. O aplicativo ira carregar em uma página princial os usuários que estão no servidor. Iniciamente quando executado pela primeira vez, veremos o indicador de carregamento quando os dados estiverem sendo buscados no servidor. E quando eles carregam, e mostrará a lista.

```python 
dependencies: 
    ...
    flutter_bloc: ^8.1.3
    http: ^1.0.0
    equatable: ^2.0.5
```
Agora vamos criar um mecanismo para obter os dados do servidor usando a API. Para poder realizar a operação http, agora criamos a pasta dentro de lib e a chamamos de repo . Dentro deste repositório, crie o novo arquivo chamado repositories.dart.

```python 
import 'dart:convert';
import 'package:http/http.dart';
import '../models/user_model.dart';

class UserRepository {
  String userUrl = 'https://reqres.in/api/users?page=2';
  //String userUrl = 'https://reqres.in/api/users?delay=3';

  Future<List<UserModel>> getUsers() async {
    Response response = await get(Uri.parse(userUrl));

    if (response.statusCode == 200) {
      final List result = jsonDecode(response.body)['data'];
      return result.map((e) => UserModel.fromJson(e)).toList();
    } else {
      throw Exception(response.reasonPhrase);
    }
  }
}

```

Para fazer a requisição http vamos usar o pacote http. Vamos dar uma olhada no nosso json agora. Estamos usando a API deste link [link](https://reqres.in/#support-heading).

Como você pode ver o formulário json no link acima, ele nos dá uma ideia de como nosso modelo deve ficar. A partir deste json vamos ter a propriedade da tag data. Aqui vamos nos concentrar no nome, sobrenome, e-mail e avatar.

Vamos construir nossa classe de modelo.

```python 
class UserModel {
  int? id;
  String? email;
  String? firstName;
  String? lastName;
  String? avatar;

  UserModel({this.id, this.email, this.firstName, this.lastName, this.avatar});

  UserModel.fromJson(Map<String, dynamic> json) {
    id = json['id'];
    email = json['email'];
    firstName = json['first_name'];
    lastName = json['last_name'];
    avatar = json['avatar'];
  }
}

```
Seguindo em frente, vamos discutir como nosso bloco funciona

![Fluxo](https://miro.medium.com/v2/resize:fit:640/format:webp/1*hj3cPU5pn2kK9OedHhDuHg.png)

Como você pode ver na foto acima.

- Primeiro temos a UI, e a partir da UI fazemos a requisição ao bloco.
- Bloco terá duas coisas, evento e estado. Primeiro, quando a interface do usuário se conecta ao bloco, ela cria e aciona o evento.
- Eventualmente, o evento chama os repositórios para o servidor por meio de um endpoint.
- Do servidor agora pegamos os dados e eles são passados ​​de volta para o bloco. À medida que temos os dados, acionamos o estado.
- Como temos a mudança no estado, a interface do usuário sabe do padrão Bloco e atualiza a interface do usuário.


Agora vamos criar uma nova pasta chamada blocs e criaremos um novo arquivo chamdo app_states.dart

```python 
import '../models/user_model.dart';
import 'package:equatable/equatable.dart';

abstract class UserState extends Equatable {}

class UserLoadingState extends UserState {
  @override
  List<Object> get props => [];
}

class UserLoadedState extends UserState {
  final List<UserModel> users;
  UserLoadedState({required this.users});
  @override
  List<Object> get props => [users];
}

class UserErrorState extends UserState {
  final String message;
  UserErrorState({required this.message});
  @override
  List<Object> get props => [message];
}
```

Equatable é usado para comparar os valores, sejam eles iguais ou não. Se duas variáveis ​​tiverem os mesmos valores, não vamos alterar ou atualizar a interface do usuário.

Primeiro, para criar o estado, precisamos criar a classe abstrata que precisa estender o equatable.

No Bloc, você deve ter em mente que todo estado é uma classe. Neste projeto temos três estados.

- Um quando os dados estão carregando.
- Segundo quando os dados são carregados.
- Em terceiro lugar, quando temos dados de busca de erro.

Após criarmos os estados,cada estado que possui evento. Agora criamos um arquivo chamado app_events.dart. Assim como state, precisamos criar uma classe para eventos.

```python
import 'package:equatable/equatable.dart';
import 'package:flutter/material.dart';

//significa que não queremos alterar as propriedades desta classe.
@immutable
abstract class UserEvent extends Equatable {
  const UserEvent();
}

class LoadUserEvent extends UserEvent {
  @override
  List<Object> get props => [];
}

```

Depois de criar o estado e o evento, precisamos encontrar a maneira de conectá-los.

Para isso precisamos criar uma nova classe e depois injetar o estado e o evento que acabamos de criar. Agora vamos em frente e conectá-los usando a biblioteca de Bloc.

Dentro da pasta blocs criamos um novo arquivo chamado app_blocs.

```python
import 'package:flutter_bloc/flutter_bloc.dart';

import '../blocs/app_states.dart';
import '../blocs/app_events.dart';
import '../repository/repositories.dart';

class UserBloc extends Bloc<UserEvent, UserState> {
  final UserRepository _userRepository;

  UserBloc(this._userRepository) : super(UserLoadingState()) {
    on<LoadUserEvent>((event, emit) async {
      emit(UserLoadingState());
      try {
        final users = await _userRepository.getUsers();
        emit(UserLoadedState(users: users));
      } catch (e) {
        emit(UserErrorState(message: e.toString()));
      }
    });
  }
}

```

Aqui temos a classe chamada UserBloc que estende o Bloc. Este Bloc deve levar e evento e estado. No nosso caso, eles são UserEvent e UserState. 

Este UserBloc possui um construtor onde passamos o repositório. Porque vamos chamar isso na interface do usuário. Conforme chamamos essa classe, passamos o UserRepository para ela. Como já usamos o Bloc precisamos passar o estado inicial para o super construtor (UserLoadingState). Depois de chamar o valor inicial, estamos chamando o método.

Agora o método on pega o tipo de evento então passamos o LoadUserEvent aqui. Você pode usar este evento para fazer algo. Quando esse evento é chamado, dizemos ei, vá em frente e emita algum estado. Nesse caso, UserLoadingState, para ver as alterações na interface do usuário.

Depois de conectar o estado, evento do nosso bloco, para acionar o estado e trabalhar com ele, criamos um novo arquivo chamado homepage.dart em nossa pasta lib.

```python
import '/blocs/app_events.dart';

import '/blocs/app_blocs.dart';
import '/repository/repositories.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

import 'blocs/app_states.dart';
import 'models/user_model.dart';

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider<UserBloc>(
          create: (BuildContext context) => UserBloc(UserRepository()),
        ),
      ],
      child: Scaffold(
        appBar: AppBar(title: const Text('BloC App')),
        body: blocBody(),
      ),
    );
  }

  Widget blocBody() {
    return BlocProvider(
      create: (context) => UserBloc(
        UserRepository(),
      )..add(LoadUserEvent()),
      child: BlocBuilder<UserBloc, UserState>(
        builder: (context, state) {
          if (state is UserLoadingState) {
            return const Center(
              child: CircularProgressIndicator(),
            );
          }
          if (state is UserLoadedState) {
            List<UserModel> userList = state.users;
            return ListView.builder(
                itemCount: userList.length,
                itemBuilder: (_, index) {
                  return Padding(
                    padding:
                        const EdgeInsets.symmetric(vertical: 4, horizontal: 8),
                    child: Card(
                        color: Theme.of(context).primaryColor,
                        child: ListTile(
                            title: Text(
                              '${userList[index].firstName}  ${userList[index].lastName}',
                              style: const TextStyle(color: Colors.white),
                            ),
                            subtitle: Text(
                              '${userList[index].email}',
                              style: const TextStyle(color: Colors.white),
                            ),
                            leading: CircleAvatar(
                              backgroundImage: NetworkImage(
                                  userList[index].avatar.toString()),
                            ))),
                  );
                });
          }
          if (state is UserErrorState) {
            return const Center(
              child: Text("Error"),
            );
          }

          return Container();
        },
      ),
    );
  }
}

```

```python
import 'package:flutter/material.dart';

import 'home_page.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Bloc',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const HomePage(),
    );
  }
}

```
## 🛠️ Consumir API com o Dio

O pacote dio é um poderoso cliente HTTP para Dart/Flutter, que suporta configuração global, interceptadores, FormData, cancelamento de solicitação, upload/download de arquivos, tempo limite e adaptadores personalizados, etc.

No nosso projeto inicialmente estamos utilizando o pacote http, caso queria deixar sua aplicação com o pacote dio, você pode fazer as seguintes mudanças. 


```python
dependencies:
  dio:
```
No arquivo repository/repositories.dart vamos comentar a função que utiliza o pacote http e criamos mais duas novas funções.

```python
 final dio = Dio();

   Future<Map<String, dynamic>> getDio() async {
    var res = await dio.get(userUrl);
    return res.data;
  }

  Future<List<UserModel>> getUsers() async {
    var json = await getDio();
    return (json['data'] as List).map((e) => UserModel.fromJson(e)).toList();
  }
```

```python

```
