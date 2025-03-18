# todolist-vue

This is a simple Todo list App using [VueJs](https://vuejs.org/)

the top icons are actions as follows :

- mark all
- unmark all
- delete a todo
- delete all todo
- import todos from raw JSON
- export todos as raw JSON
- add a todo (set the name first, description is optional)

the format importing format is like this

```json
[
  {
    "title": "an unmarked todo",
    "status": false
  },
  {
    "title": "an unmarked todo with a description",
    "description": "the description",
    "status": false
  },
  {
    "title": "a marked todo",
    "status": true
  }
]
```

the todo list used in the making of this app
```json
[{"title":"Display the first todo","status":true},{"title":"Update a todo's status","status":true},{"title":"Remove a todo","status":true},{"title":"Add a todo","status":true},{"title":"Save todos to localstorage","status":true},{"title":"export todos","status":true},{"title":"import todos","status":true},{"title":"Publish a demo on github pages from github actions","status":true}]
```
