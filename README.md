


# Opal WebSocket

`Opal WebSocket` es una librería escrita en **Crystal** para gestionar conexiones WebSocket de manera sencilla. Permite manejar eventos personalizados, rutas de WebSocket, y callbacks para eventos como apertura, recepción de mensajes y cierre de conexión.

## Características

- Gestión de eventos personalizados en WebSockets.
- Rutas WebSocket configurables para diferentes endpoints.
- Soporte para callbacks en eventos `on_open`, `on_message`, y `on_close`.
- Arquitectura fácil de escalar.

## Instalación

### Requisitos previos

- **Crystal**: Asegúrate de tener [Crystal](https://crystal-lang.org/install/) instalado en tu sistema.


### Instrucciones de instalación

1. **Instala la librería**:

   Si usas **Shards** (gestor de dependencias de Crystal), agrega la librería a tu archivo `shard.yml`:

   ```yml
   dependencies:
     opal-web-socket:
       github: Stockers-JAPG/opal-web-socket
   ```

2. **Instala las dependencias**:

   Luego, ejecuta el siguiente comando para instalar las dependencias:

   ```crystal
   shards install
   ```

## Uso

### Inicialización

Para utilizar la librería, solo necesitas importar los módulos relevantes y definir las rutas WebSocket junto con los callbacks correspondientes.

```crystal
require "opal-web-socket"

router = Opal::WebSocket::Router.new

# Definir un handler para el WebSocket
handler = Opal::WebSocket::Handler.new(
  on_open: ->(socket : HTTP::WebSocket) do
    puts "Conexión WebSocket abierta: #{socket.object_id}"
  end,
  on_message: ->(socket : HTTP::WebSocket, message : String) do 
     puts "Mensaje recibido: #{message}" 
  end,
  on_close: ->(socket : HTTP::WebSocket) do 
    puts "Conexión cerrada" 
  end
)

# Configurar las rutas WebSocket
router.websocket("/chat", handler)

# Crear el servidor HTTP y configurar WebSocket
server = HTTP::Server.new do |context|
  if router.call(context)
    # El WebSocket está manejado por el router
  else
    context.response.content_type = "text/plain"
    context.response.print "¡Hola Mundo!"
  end
end

server.bind_tcp("0.0.0.0", 8080)
puts "Servidor corriendo en http://0.0.0.0:8080"
server.listen
```

### Explicación del Código

1. **Definición de Handlers**: 
   - El `Handler` es donde defines cómo manejar los eventos del WebSocket. Los eventos incluyen `on_open`, `on_message` y `on_close`.
   
2. **Definición de Rutas WebSocket**:
   - El `Router` gestiona las rutas WebSocket y las asocia a los `Handler` adecuados. En este ejemplo, hemos configurado una ruta `/chat`.

3. **Servidor HTTP**:
   - El servidor HTTP maneja tanto las conexiones HTTP estándar como las conexiones WebSocket, dirigiendo estas últimas a través del `Router`.

### Eventos Personalizados

Puedes registrar y disparar eventos personalizados usando el módulo `Opal::WebSocket::Events`.

#### Registrar un evento

```crystal
Opal::WebSocket::Events.on("mensaje_enviado") do |socket, data|
  puts "Mensaje recibido desde el cliente: #{data}"
end
```

#### Disparar un evento

```crystal
Opal::WebSocket::Events.trigger_event("mensaje_enviado", socket, JSON.parse({ "saludo" => "Hola desde el servidor" }.to_json))
```

### Rutas WebSocket

Puedes definir rutas WebSocket con parámetros dinámicos utilizando `:` en la ruta.

```crystal
router.websocket("/chat/:room_id", handler)
```

El `Router` se encargará de extraer el valor de `room_id` de la URL y proporcionarlo en los parámetros cuando sea necesario.

### Ejemplo Completo

```crystal
require "opal-web-socket"

# Definir los handlers
handler = Opal::WebSocket::Handler.new(
  on_open: ->(socket : HTTP::WebSocket) do
    puts "Conexión abierta"
  end,
  on_message:->(socket : HTTP::WebSocket, message : String) do
    puts "Mensaje recibido: #{message}"
    Opal::WebSocket::Events.trigger_event("mensaje_enviado", socket, JSON.parse(message))
  end,
  on_close: ->(socket : HTTP::WebSocket) do
    puts "Conexión cerrada" 
  end
)

# Configurar las rutas WebSocket
router = Opal::WebSocket::Router.new
router.websocket("/chat/:room_id", handler)

# Servidor HTTP
server = HTTP::Server.new do |context|
  if router.call(context)
    # Si la solicitud es WebSocket, el router maneja la conexión.
  else
    context.response.content_type = "text/plain"
    context.response.print "¡Hola Mundo!"
  end
end

server.bind_tcp("0.0.0.0", 8080)
puts "Servidor corriendo en http://0.0.0.0:8080"
server.listen
```


## API

### `Opal::WebSocket::Events`

- `crystal on(event_name : String, &block : Proc(HTTP::WebSocket, JSON::Any, Nil))`:
  - Registra un evento personalizado con un bloque de código para ser ejecutado cuando se dispare ese evento.
  
- `trigger_event(event_name : String, socket : HTTP::WebSocket, data : JSON::Any)`:
  - Dispara un evento previamente registrado con los datos especificados.

### `Opal::WebSocket::Handler`

- `on_open: ->(socket : HTTP::WebSocket)`:
  - Callback opcional que se ejecuta cuando un WebSocket se abre.

- `on_message:->(socket : HTTP::WebSocket, message : String)`:
  - Callback opcional que se ejecuta cuando se recibe un mensaje.

- `on_close: ->(socket : HTTP::WebSocket)`:
  - Callback opcional que se ejecuta cuando un WebSocket se cierra.

### `Opal::WebSocket::Router`

- `websocket(path : String, handler : Handler)`:
  - Registra una ruta WebSocket con su respectivo `Handler`.

- `call(context : HTTP::Server::Context) : Bool`:
  - Llama al router para gestionar una solicitud HTTP, redirigiéndola si es un WebSocket.

## Contribuir

1. Forkea el repositorio.
2. Crea una nueva rama (`git checkout -b feature-nueva-caracteristica`).
3. Realiza los cambios y haz commits.
4. Envía un pull request con una descripción clara de los cambios.

## Contributors

- [Jose Antonio Padre Garcia](https://github.com/Stockers-JAPG) - creator and maintainer