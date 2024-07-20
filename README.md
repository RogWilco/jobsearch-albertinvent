# Albert Notebook

## Instructions

### Abstract
Albert Notebook enables users to create free-flow content using various notebook components, including text, images, chemical drawings, mentions, and files, with plans to support Albert entities in the future. The notebook also supports content reordering and maintains a version history.

### Exercise (45 Min)
Design a backend for the Notebook (API and database schema) to implement the specified functionality. Utilize OpenAPI 3.0 to define all REST endpoints. Use DynamoDB with a single table design to store and retrieve transactional data. The API should be optimized for patching the notebook, retrieving version history, and reordering notebook content.

## Design

### Considerations

#### Entities

##### Primary Entities

The core entities involved in the previously identified access patterns.

```typescript
type Notebook = {
  id: CUID
  title: string
  revisions: Revision[]
}

type Revision = {
  id: CUID
  elements: Element[]
  createdAt: DateTime
  createdBy: User
}

type Element = {
  id: CUID
  type: 'text' | 'image' | 'mention' | 'file'
  order: number
  value: string | Object
}
```

##### Secondary Entities

Candidates for additional tables or other persistence mechanisms.

```typescript
type Organization = {
  name: string
  notebooks: Notebook[]
  users: User[]
}

type User = {
  name: string
  customer: Customer
}
```

#### Access Patterns

- **Patching the notebook:**
  > For a given notebook, create a new revision based on the previous with the specified partial changes.

  ```PATCH /notebooks/:id``` \
  ```POST /notebooks/:id/revisions``` (alias)

- **Retrieving version history:**
  > For a given notebook, fetch a list of revisions.

  ```GET /notebooks/:id/revisions```

- **Reorder notebook content:**
  > For a given notebook, create a new revision based on the previous, with its content reordered.

  ```PATCH /notebooks/:id```

#### Future Access Patterns
- **Creating a new notebook:**
  > Create a new notebook with an initial revision.

  ```POST /notebooks```

- **Retrieving the latest version of a notebook:**
  > For a given notebook, fetch the latest revision.

  ```GET /notebooks/:id``` \
  ```GET /notebooks/:id/revisions/latest```

- **Retrieving a specific version of a notebook:**
  > For a given notebook, fetch a specific revision.

  ```GET /notebooks/:id/revisions/:revisionId```

- **Reverting to a specific version of a notebook:**
  > For a given notebook, revert to a specific revision.

  ```??? ```

### API

`PATCH /api/v1/notebook/:id` (alias: `POST /api/v1/notebook/:id/revisions`)

### Persistence

#### Single-Table Design

In this case, a single table design would encompass all the data relevant to the
identified access patterns. The table would have a composite primary key with the
`notebookId` as the partition key and the `revisionId` as the sort key. The `elements`
attribute would be a list of maps, each map representing an element in the notebook.

```json
{
  "notebookId": "UUID",
  "revisionId": "UUID",
  "elements": [
    {
      "id": "UUID",
      "order": 1,
      "type": "text",
      "value": "Hello, World!"
    },
    {
      "id": "UUID",
      "order": 2,
      "type": "image",
      "value": "https://example.com/image.jpg"
    }
  ]
}
```
