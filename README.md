# 🤖 Poe for Home Assistant

<a name="installing"></a>
## Installation

#### Method 1: [HACS (**Click to install**)](https://my.home-assistant.io/redirect/hacs_repository/?owner=ichuixue&repository=poe-chat&category=integration)

#### Method 2: Manually install via Samba / SFTP
> [Download](https://github.com/ichuixue/poe-chat/archive/main.zip) and copy `custom_components/poe_chat` folder to `custom_components` folder in your HomeAssistant config folder

#### Method 3: Onkey shell via SSH / Terminal & SSH add-on
```shell
wget -q -O - https://hacs.vip/get | HUB_DOMAIN=ghproxy.com/github.com DOMAIN=poe_chat REPO_PATH=ichuixue/poe-chat ARCHIVE_TAG=main bash -
```

#### Method 4: shell_command service
1. Copy this code to file `configuration.yaml`
    ```yaml
    shell_command:
      update_poe_chat: |-
        wget -q -O - https://hacs.vip/get | HUB_DOMAIN=ghproxy.com/github.com DOMAIN=poe_chat REPO_PATH=ichuixue/poe-chat ARCHIVE_TAG=main bash -
    ```
2. Restart HA core
3. Call this [`service: shell_command.update_poe_chat`](https://my.home-assistant.io/redirect/developer_call_service/?service=shell_command.update_poe_chat) in Developer Tools


## Config

1. `name`: Config entry name, unique
2. `token`: Poe token, `p-b` in cookies

   Find Your Poe Token: Log into [Poe](https://poe.com) on any desktop web browser, then open your browser's developer tools (also known as "inspect") and look for the value of the `p-b` cookie in the following menus:
   - Chromium: Devtools > Application > Cookies > poe.com
   - Firefox: Devtools > Storage > Cookies
   - Safari: Devtools > Storage > Cookies
4. `proxy`: Proxy to use, `http://192.168.xx.xx:7890`

## Siri Shortcut for Apple devices

1. Download the [Nod-Red flow for Siri Shortcut](https://github.com/ichuixue/poe-chat/blob/main/poe-chat_siri-shortcut_flows.json) and added it to Node-Red in your homeassistant.
2. Download the [latest version of Poe-Chat Siri Shortcut](https://www.icloud.com/shortcuts/3938233c9bff4c53b29e632c841e3003) on your iPhone or iPad and Config the relevant parameters according to the pop-up window during installation.
   - `HA_address`: Your homeassistant IP_ADDRESS, e.g., `http://HA_IP_ADDRESS:8123`
   - `HA_token`: `The Long-Lived Access Token` obtained at `http://HA_IP_ADDRESS:8123/profile`
   - `entry_name`: The entry name when you config the `Poe Chat` integration
   - `bot_name`: The default bot you want to use. Please refer to the corresponding relationship below:
     ```
     {
      "capybara": "Sage",
     "a2": "Claude-instant",
     "nutria": "Dragonfly",
     "a2_100k": "Claude-instant-100k",
     "beaver": "GPT-4",
     "chinchilla": "ChatGPT",
     "a2_2": "Claude+"
     }
     ```
4. Run the Siri shortcut once and agree to all permissions.
5. Now you can use `hey siri 对话` to talk with ChatGPT on Apple devices (iPhone/iPad/Apple Watch/Homepod...)

## HA service

- [![Call service: poe_chat.chat](https://my.home-assistant.io/badges/developer_call_service.svg) `poe_chat.chat`](https://my.home-assistant.io/redirect/developer_call_service/?service=poe_chat.chat)
  ```yaml
  service: poe_chat.chat
  data:
      name: poe # Config entry name
      bot: capybara
      message: Hello
      conversation_id: xxxx
      throw: false      # Output reply result to HA notifications
      throw_chunk: true # Output chunked reply text to HA notifications
      extra: # Optional
          chunk_size: 128
          chunk_mark: true
          chunk_code: true
  ```

### Event

- `poe_chat.reply`
- `poe_chat.reply_chunk`
  ```yaml
  event_type: poe_chat.reply
  data:
    name: poe
    bot: capybara
    message: Hello
    conversation_id: xxxx
    id: TWVzc2FnZTozMjM1OTk4Nzc=
    messageId: 323599877
    creationTime: 1681373526812436
    state: incomplete
    author: capybara
    text: Hello! How can I assist you today?
    linkifiedText: Hello! How can I assist you today?
    text_new: assist you today?
    suggestedReplies: []
  ```
- `poe_chat.reply_error`
  ```yaml
  event_type: poe_chat.reply_error
  data:
    name: poe
    bot: capybara
    message: Hello
    conversation_id: xxxx
    error: xxxx
  ```

### Example

- Chat with Xiaoai speaker via [`hass-xiaomi-miot`](https://github.com/al-one/hass-xiaomi-miot)
  ```yaml
  alias: Chat with Xiaoai speaker
  trigger_variables:
    conversation_id: xiaoai_chat
  trigger:
    - platform: state
      entity_id: sensor.xiaomi_x08c_xxxx_conversation
      id: send
    - platform: event
      event_type: poe_chat.reply
      event_data:
        conversation_id: "{{ conversation_id }}"
      id: reply
  condition: []
  action:
    - choose:
        - conditions:
            - condition: trigger
              id: send
            - condition: template
              value_template: |-
                  {% set sta = trigger.to_state.state|default('') %}
                  {{ sta|regex_findall('给我|请问|告诉我')|length > 0 }}
          sequence:
            - service: xiaomi_miot.intelligent_speaker
              data:
                entity_id: media_player.xiaomi_x08c_xxxx
                text: 闭嘴
                execute: true
                silent: true
            - service: poe_chat.chat
              data:
                name: poe # Config entry name
                bot: chinchilla
                message: "{{ trigger.to_state.state }}"
                conversation_id: "{{ conversation_id }}"
        - conditions:
            - condition: trigger
              id: reply
          sequence:
            - service: xiaomi_miot.intelligent_speaker
              data:
                entity_id: media_player.xiaomi_x08c_xxxx
                text: "{{ trigger.event.data.text }}"
                execute: false
  mode: queued
  max: 10
  ```


## Thanks

- https://github.com/ading2210/poe-api
