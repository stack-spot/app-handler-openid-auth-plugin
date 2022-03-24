# About
## OpenId Auth Plugin 

Plugin que adiciona a capacidade autenticação baseada em OpenId a stack, criando assim uma lambda que faz essa integração e torna disponível a autenticação dos endpoints ao adicionar uma configuração das operações descritas no contrato OpenAPI responsável por criar os endpoints.

Exemplo:

```yaml
    get:
      operationId: get-auction
      description: Get auction data by id

      # operation level
      security:
        - jwtAuth: []
```




# Implementation

## Enabling JWT Security

- The construct supports JWT tokens for security and the security can be enabled defining the following [OpenAPI security scheme](https://swagger.io/specification/#security-scheme-object):

```yaml
components:
  securitySchemes:
    jwtAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

- You can enable JWT token validation for all operations defining a securty constraint at api root level or at operation level as shown below:

```yaml
# root level
security:
  - jwtAuth: []

paths:
  /auctions/{id}:
    parameters:
      - $ref: '#/components/parameters/AuctionId'
      - $ref: '#/components/parameters/Authorization'
    get:
      operationId: get-auction
      description: Get auction data by id

      # operation level
      security:
        - jwtAuth: []

      tags:
        - Auction services
      responses:
        '200':
          description: Auction returned successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Auction'
        '404':
          description: Auction not found
```

- Most IDM providers expose a JWKS_URI with their public keys to verify JWT token signatures. You need to configure the construct as shown below to inform JWKS_URI to be used to get the public keys:

```typescript
const api = new StackSpotOpenApiServices(this, 'StackSampleAPI', {
  specPath: 'spec/auction-api.yaml',
  jwksUri: 'https://some.idm.provider/auth/realms/some-realm/protocol/openid-connect/certs',
});
```

- If you are using an IDM that supports OpenID connect you can get JWKS_URI endpoint in [well-known endpoint](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig) of OpenID connect provider.

- When you enable JWT authorization your controllers will extend `JWTAuthorizationController` class and you can override the `authorizeResourceAccess` method to do some custom authorization logic. The JWT token payload can be accessed using `this.jwtTokenPayload` protected property. Controlles already generated before JWT authorization will not be overwrited an must be changed by the user.

- With JWT Authorization enabled an API Gateway Lambda authorizer will be configured to validate the token.

- Authorization logic is not made by this lambda only the authenticity and validity of token is verified. You need to implement your authorization logic using token claims in operations controllers or create a new base constroller class based on `JWTAuthorizationControler` to use as base class of your controllers and implement authorization logic on it.