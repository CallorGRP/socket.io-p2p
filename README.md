![](https://cldup.com/95U80xyuHq.svg)

이 모듈은 피어 간의 WebRTC 연결을 설정하고 이벤트 (socket.io-protocol)를 사용하여 통신하는 **쉽고** **안정적인** 방법을 제공합니다.([socket.io-protocol](https://github.com/Automattic/socket.io-protocol))

Socket.IO는 [신호 데이터](http://www.html5rocks.com/en/tutorials/webrtc/infrastructure/#what-is-signaling)를 전송하고 WebRTC `PeerConnection`이 지원되지 않는 클라이언트의 대체 수단으로 사용됩니다.

## 사용하는 방법

소켓 연결을 만들고 P2P에 전달합니다.
클라이언트코드 :

```js
var P2P = require("socket.io-p2p");
var io = require("socket.io-client");
var socket = io();

var p2p = new P2P(socket);

p2p.on("ready", function () {
  p2p.usePeerConnection = true;
  p2p.emit("peer-obj", { peerId: peerId });
});

// this event will be triggered over the socket transport
// until `usePeerConnection` is set to `true`
p2p.on("peer-msg", function (data) {
  console.log(data);
});
```

browserify를 사용하지 않는 경우 포함 된 독립 실행 형 파일 `socketiop2p.min.js`를 사용하십시오.  
이것은 `window`에 `P2P` 생성자를 내 보냅니다.

서버에서 [socket.io-p2p-server](https://github.com/tomcartwrightuk/socket.io-p2p-server)를 사용하여 신호를 처리하십시오.  
WebRTC 데이터 연결을 지원하는 모든 클라이언트는 기본 `/` 네임 스페이스를 통해 신호 데이터를 교환합니다.

```js
var server = require("http").createServer();
var io = require("socket.io")(server);
var p2p = require("socket.io-p2p-server").Server;
io.use(p2p);
server.listen(3030);
```

WebRTC 피어 연결은 socket.io 룸 내에서 신호 데이터를 교환하여 설정할 수도 있습니다.  
`connection` 콜백 내에서 `p2p` 서버를 호출하면됩니다.

```js
var server = require("http").createServer();
var io = require("socket.io")(server);
var p2p = require("socket.io-p2p-server").Server;
server.listen(3030);

io.on("connection", function (socket) {
  clients[socket.id] = socket;
  socket.join(roomName);
  p2p(socket, null, room);
});
```

전체 예제 : [chat app](https://github.com/socketio/socket.io-p2p/tree/master/examples/chat)

## API

### `p2psocket = new P2P(socket, opts, cb)`

새로운 `socket.io-p2p` 연결을 만듭니다.

The `opts` object can include options for setup of the overall socket.io-p2p connection as well as options for the individual peers - specified using `peerOpts`.

`opts` 객체는 `peerOpts`를 사용하여 지정된 개별 피어에 대한 옵션뿐만 아니라  
전체 socket.io-p2p 연결 설정 옵션을 포함 할 수 있습니다.

- `numClients` - 각 클라이언트가 연결할 수있는 최대 피어 수, 기본값은 `5`입니다.
- `autoUpgrade` - 피어가 준비되면 s.io에서 p2p 연결로 업그레이드 합니다. 기본값은 `true` 입니다
- `peerOpts` - 기본 피어에 전달할 옵션 개체입니다. 현재 지원되는 옵션은 [여기](https://github.com/feross/simple-peer/blob/master/README.md#api)를 참조하십시오. 예를 보려면 [여기](examples/streaming)를 참조하십시오.

`cb`는 선택적 콜백입니다. 연결이 WebRTC 연결로 업그레이드 될 때 호출됩니다.

### `p2psocket.on('upgrade', function (data) {})`

P2P 연결이 WebRTC 연결로 변환 될 때 트리거됩니다.

### `p2psocket.on('peer-error', function (data) {})`

신호를 보내는 동안 `PeerConnection` 개체 오류가 발생하면 트리거됩니다.

## 개발 로드맵

- 여러 이진 Blob을 포함하는 패킷 blobs 패킷은 이 버전에서 하나의 Blob 만 포함 할 수 있습니다
- 피어가 PeerConnection을 지원하지 않는 피어와 지원하는 피어간에 릴레이 역할을 할 수 있도록 허용합니다.

PR 및 문제 보고서를 환영합니다.
