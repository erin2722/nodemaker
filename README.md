# Nodemaker

Proof of concept for n8n node generator.

Functionality:
- Generate main node files: `*.node.ts` and `GenericFunctions.ts`
- Generate extra node files: `*Description.ts`
- Generate node credentials file: `*.credentials.ts`
- Generate an updated `package.json`
- Generate docs for node: `*.ts` → coming soon!
- Place generated files in the n8n repo.
- Place generated docs in the n8n docs repo.

## Installation

1. Clone repo.
2. Install dependencies: `npm i`
3. Ensure nodemaker is alongside the n8n and n8n docs repos.

```bash
.
├── n8n
├── n8n-docs
└── nodemaker
```

## Operation

1. Enter node params in `parameters.js`. (This file is a functional example.) More info on editing params below.

2. Run an operation.

```
$ npm run [script]
```

| Script   | Action                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| `nodegen --simple` | Generate `*.node.ts` and `GenericFunctions.ts`                                                       |
| `nodegen --complex`    | Generate `*.node.ts` and `GenericFunctions.ts` and place resources in multiple `*Description.ts` files.           |
| `packgen`   | Generate an updated `package.json`                                             |
| `empty`    | Clear the `/output` dir. |
| `place --node`    | Move the files in `/output` to their appropriate locations in the n8n repo. |
| `place --docs`    | Move the files in `/output/docs` to their appropriate locations in the n8n docs repo. |


### Notes

- All output files are generated in the `/output` dir. Same-name files are overwritten.
- When node files are placed in the n8n repo, `output/package.json` will overwrite `/packages/nodes-base/package.json`.
- The `package.json` used for file generation is retrieved at runtime from the official repo.
- No credential file will be generated if `metaParameters.auth` is an empty string.
- Some generated files contain `// TODO:` lines, for the developer to add in custom logic.

## Editing params

Node params are divided into:
- `metaParameters` (service-related)
- `mainParameters` (resource-related)

Types for both sets of parameters are defined in `globals.d.ts`. Wrong parameters trigger TypeScript error messages.

### `metaParameters`

```js
const metaParameters = {
  serviceName: "Some Service", // casing and spacing as in original service
  auth: "OAuth2",
  nodeColor: "#ff6600",
  apiUrl: "http://api.service.com/",
};
```

### `mainParameters`

`mainParameters` is a big object containing resource names as properties pointing to arrays of operations.

Operations all have the same five fields `name`, `description`, `endpoint`, `requestMethod` and `fields`. `endpoint` can contain a variable between double dollar signs, as in `endpoint: "items/$$articleId$$"`, to allow for the endpoint to be fully autogenerated.

The operation's `fields` property points to an array of objects, which may be:

- **Regular fields**, which contain the properties `name`, `description`, `type`, and `default`, or
- **Group fields**, which contain the properties `name`, `description`, `type: collection` or `type: multiOptions`, `default: {}`, and `options`.

Group fields with `name: "Additional Fields"` should be used for optional fields, i.e. those not needed for the request to succeed.

The `options` property points to an array of objects structured like a regular or group field. Two-level nesting is supported. When using `multiOptions`, `options` should point to an object containing only `name` and `description`.

```js
const mainParameters = {
    Entity1: [
        // first operation for Entity1 (some resource)
        {
            name: "Get",
            description: "Get some entity",
            endpoint: "entity/$$entityId$$",
            requestMethod: "GET",
            fields: [
                {
                    name: "Entity ID",
                    description: "The ID of entity to be returned",
                    type: "string",
                    default: "",
                },
                {
                    name: "Additional Fields",
                    type: "collection",
                    default: {},
                    options: [
                        {
                            name: "Keyword",
                            description: "The keyword for filtering the results of the query",
                            type: "string",
                            default: "",
                        },
                    ],
                },
            ],
        },
        // second operation for Entity1 (same resource as above)
        {
            name: "Put",
            // etc.
        }
    ],
    Entity2: [
        // etc.
    ],
}
```

Field display restrictions are inferred from the object structure, but if you have an additional field display restriction, you can add it with `extraDisplayRestriction: { fieldName: boolean }`.
