# architecture-diagrams

## Login

::: mermaid
sequenceDiagram
    buyer->>+react ambassador: Access to reactambassador.com/login
    react ambassador ->>-buyer: Render login form
    buyer->>+react ambassador: Fill form and press Sign in button
    react ambassador->>+api gateway: send login data to backend.com/login
    api gateway->>+auth microservice: send login data to auth.com/login
    alt correct credentials
        auth microservice->>api gateway: send token to client
    else incorrect credentials
        auth microservice->>api gateway: send error
    end
    api gateway->>-react ambassador: send response
    alt is token
        react ambassador->>buyer: redirect reactambassador.com/
    else is error
        react ambassador->>buyer: render error message
    end
:::

## Register

::: mermaid
sequenceDiagram
    buyer->>react ambassador: Access to reactambassador.com/register
    react ambassador->>buyer: Render register form
    buyer->>react ambassador: Fill form and press Submit button
    react ambassador->>api gateway: send user data to backend.com/register
    api gateway->>user microservice: send user data to user.com/register
    user microservice->>user db: save user
    user db->>user microservice: return user saved
    user microservice->>+circuit breaker: send lite user data to circuitbreaker.com/user
    alt is open state
        circuit breaker->>-user microservice: return error response
    else is closed | half open
        circuit breaker->>+auth microservice: send lite user data to auth.com/user
        alt is timeout
            circuit breaker->>user microservice: return error response
        else is not timeout
            auth microservice->>auth db: save lite user
            auth db->>auth microservice: return lite user saved
            auth microservice->>circuit breaker: return lite user saved
            circuit breaker->>user microservice: return lite user saved
        end
    end
    user microservice->>api gateway: return response
    api gateway->>react ambassador: return response
    alt creaded successfully
        react ambassador->>buyer: redirect reactambassador.com/login
    else is error
        react ambassador->>buyer: render error message
    end
:::