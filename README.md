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
Enfin, il est nécessaire d'implémenter le *provider* qui gère tout ça. Toujours par comparaison avec les autres *provider* et à nouveau par analyse de l'interface UseCase, on comprend qu'il faut retourner la réponse de la base de donnée sans distinction entre le cas **create** et le cas **update**. La seule subtilité est qu'on demande de valider les données avant. Puisqu'on s'attend à trouver un objet de type `Task`, on vérifie que l'objet contienne au moins un attribut `name` ou un attribut `id`, sinon l'interaction avec la database ne sert à rien. L'implémentation est donc :

```ts
try {
  const { id, name } = dto

  if (!id && !name) return null;
  
  return this.taskRepository.save(dto);
} catch (error) {
  throw new BadRequestException(error.message);
}
```
