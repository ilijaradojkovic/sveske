# CometChat API

### Get all chats for user

Conversations-> List All

https://23727500d0acf346.api-eu.cometchat.io/v3/conversations?perPage=100&page=1

moramo da ubacimo header  `onBehalfOf ` za kog user-a





### Details of a chat

Messages->List User Spesific Messages

https://23727500d0acf346.api-eu.cometchat.io/v3/users/superhero1/messages

Moramo da ubacimo i  `onBehalfOf ` u header-u





### Delete chat

Conversations-> Delete

https://23727500d0acf346.api-eu.cometchat.io/v3/users/superhero1/conversation

moramo da ubacimo u body:

znaci jer u url-u imamo superhero1 a u body superhero2 on ce da izbrise sve a njih

```json
{
  "deleteMessagesPermanently": true,
  "conversationWith": "superhero2"
}
```





### Send Message

Messages->Send

 https://23727500d0acf346.api-eu.cometchat.io/v3/messages

u heder-u onBehalfOf to je ko salje

u body:

```
{
  "category": "message",
  "type": "text",
  "data": {
    "text": "cao1212"
  },
  "receiver": "superhero1",
  "receiverType": "user"

}
// reciver type -> user ,group
// category -> custom,message
// type -> text,image,file,audio,video
// data -> ima text,metadata,customData,attachments
// onBehalfOf -> od koga je ,default je system,OVO JE U HEADER-U
```

Auth token pravimo ako hocemo da damo klijentu dozvolu kao oauth2 da on radi,da mi ne leak nas api key nego smo pravimo tokene