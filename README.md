# Solution de l'exercice

## TODO : TaskController
Les premiers todo consistent à complémenter les controllers par mimétisme de ceux déjà existant. La première chose à faire est d'aller voir dans la structure du projet quel `provider` correspond aux routes `create` et `update`. On trouve alors la classe `SaveTaskUseCase` qui n'est pas implémentée non plus. Ce n'est pas important à ce stade. En regardant l'interface générique `UseCase` de plus près, ainsi que la signature des *controllers* et du *provider* associé, on comprend que la seule implémentation valide est :

```ts
return (await this.useCaseFactory.create(SaveTaskUseCase)).handle(dto);
```

A noter que pour que cela fonctionne, il faut également ajouter le type `SaveTaskUseCase` à l'interface `UseCase` :

```ts
type UseCases = GetAllTasksUseCase | DeleteTask | SaveTaskUseCase;
```

## TODO : TaskRepository
La deuxième série de todo concerne l'implémentation du *repository pattern*. On voit que la fonction `save(dto)` est prévue pour traiter le cas **create** et le cas **update**. En l'absende d'`id`, on considère qu'une nouvelle tâche est créée, ce qui donne l'implémentation :

```ts
if (!data.id) {
  // @todo IMPLEMENT HERE USING PRISMA API
  return this.prisma.task.create({
    data: data as Prisma.TaskCreateInput
  });
}
```

Où le cast sur la variable `data` est nécessaire pour faire comprendre au transpilateur que l'objet ne contient pas d'`id`.

Pour le cas où un `id` est fourni, on modifie la tâche identifié :

```ts
return this.prisma.task.update({
  where: { id: Number(data.id) },
  data: data,
})
```

## TODO : SaveTaskUseCase
Enfin, il est nécessaire d'implémenter le *provider* qui gère tout ça. Toujours par comparaison avec les autres *provider* et à nouveau par analyse de l'interface UseCase, on comprend qu'il faut retourner la réponse de la base de donnée sans distinction entre le cas **create** et le cas **update**. Il est demandé de valider les données. Cependant, la vérification de l'`id` se fait déjà dans la fonction `save(dto)` et le frontend est pensé pour d'abord créer une tâche vide puis lui donner un nom en la modifiant. Ainsi, il n'est pas nécessaire de vérifier que l'objet reçu `dto` contient au moins un des attributs `id` ou `name`. On peut laisser l'erreur se propager. L'implémentation est donc :

```ts
try {
  return this.taskRepository.save(dto);
} catch (error) {
  throw new BadRequestException(error.message);
}
```

Il fallait également ajouter le repository au *provider* :

```ts
constructor(private readonly taskRepository: TaskRepository) {}
```
