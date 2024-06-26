# Notification Badges for Rooms on the HA Dashboard
<image src="https://github.com/satyambaba/homeassistant/assets/6833101/6c48fd45-f4d4-45ab-aed0-603ce5eaeb79" width="500" />

<br><br>
**1. Introduction**

I have created a "notification" system for different rooms on my HA dashboard. It displays useful info which can indicate what's going on in a quick glance. Numbers and icons denote "what" and colors represent whether it is an "info" (blue), a good thing (green) or something that needs my attention (red). I had made it a long time ago and it proved to be very helpful at several occasions.
For example (in the picture above), here are the things which can be inferred in just one quick glance - 
- Bedroom thermostat is set to cooling
- My washing machine is running
- My aquarium needs my attention (I have deliberately turned off the filter, so that this can be shown)
- My gaming system is currently on
- Two security cameras are on
- Three devices are either offline or they are low on batteries.


**2. Approach**

It is much simpler than it looks. Template sensors are created for the info that are needed which contains the "content ([icons](https://www.emojipedia.org), text or numbers)" and severity (blue - info, green - a good thing, red - needs attention) 


**3. Requirements**
- [Vertical Stack In Card](https://github.com/ofekashery/vertical-stack-in-card)
- [Paper Buttons Row Card](https://github.com/jcwillox/lovelace-paper-buttons-row)
- [Card Mod](https://github.com/thomasloven/lovelace-card-mod)


**4. Steps to Follow**

**A. Edit your theme file and create few variables**
```yaml
  notification-badge-green: "#149c14"
  notification-badge-red: "#ff0000"
  notification-badge-neutral: "#3765ab"
```
Please don't forget to reload your theme using service: frontend.reload_themes


**B. Create a script called dummy which does nothing, it just acts as a placeholder.**
```yaml
dummy:
  alias: Dummy
  sequence:
    delay:
      milliseconds: 1
```


**C. Create template sensors for each room. Here. I'll create two sensors for the laundry room and security page for reference.**

For Laundry Room - Badge to denote if the washing machine is on
```yaml
    laundry_room_notification:
      friendly_name: "Laundry Room Notification"
      value_template: >
        {% if states('input_boolean.washing_machine_running') == 'on'%}👚_N
        {% else %}{% endif %}
```
Plaese pay attention to 👚_N. The first part is an icon, it can also be a number, a text or a dynamic template. Please note the "_N" is for neutral badges (blue). For good badges, use _G and for badges that need your attention, use _R.

A good source for the icons are [emojipedia.org](https://www.emojipedia.org)

For Security Page - Badge to denote how many security cameras are on + if my bird camera is off + if my doorbell is offline
```yaml
    security_notification:
      friendly_name: "Security Notification"
      value_template: >
        {% if states('switch.bird_camera_switch') in ['off','unavailable'] %}!_R
        {% elif states('binary_sensor.doorbell_ringing') in ['unavailable'] or states('binary_sensor.doorbell_connected') == 'off' %}🔔_R
        {% elif states('sensor.on_camera_count') | int > 0 %}
          {{expand('group.all_cameras')|selectattr('state','in',['on'])|list|count}}_G
        {% else %}{% endif %}
```
Here you can clearly see, this can be pretty dynamic. If my bird camera is off, it will show "!" in red or if my doorbell is offline, it will show a bell icon in red. If both of these cases are not true, it will simply show the number of cameras that are on in green. And, if none of these conditions are true, it will show nothing.


**D. Create a navigation panel.**

Here I am providing the code for my navigation panel. You can customize it as per your need. Please focus on the "name" and the "background" part for each room in this panel. Possibilities are endless, just use your imagination and have fun.
```yaml
view_layout:
  column: 1
type: custom:vertical-stack-in-card
card_mod:
  style: |
    ha-card{
      background: var(--nav-background-color);
      box-shadow: var(--nav-box-shadow);
    }
cards:
  - type: custom:paper-buttons-row
    buttons:
      - entity_id: script.dummy
        icon: mdi:home
        name: |
          {{states('sensor.home_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: |
              {% if states('sensor.home_notification').split('_')[1] == 'R' %}
                var(--notification-badge-red)
              {% elif states('sensor.home_notification').split('_')[1] == 'G' %}
                var(--notification-badge-green)
              {% elif states('sensor.home_notification').split('_')[1] == 'N' %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/home
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:bed
        name: |
          {{states('sensor.bedroom_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.bedroom_notification').split('_')[1] == 'R'
              %}
                var(--notification-badge-red)
              {% elif states('sensor.bedroom_notification').split('_')[1] == 'G'
              %}
                var(--notification-badge-green)
              {% elif states('sensor.bedroom_notification').split('_')[1] == 'N'
              %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/bedroom
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: hue:room-living
        name: |
          {{states('sensor.living_room_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.living_room_notification').split('_')[1] ==
              'R' %}
                var(--notification-badge-red)
              {% elif states('sensor.living_room_notification').split('_')[1] ==
              'G' %}
                var(--notification-badge-green)
              {% elif states('sensor.living_room_notification').split('_')[1] ==
              'N' %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/living-room
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:pot-steam
        name: |
          {{states('sensor.kitchen_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.kitchen_notification').split('_')[1] == 'R'
              %}
                var(--notification-badge-red)
              {% elif states('sensor.kitchen_notification').split('_')[1] == 'G'
              %}
                var(--notification-badge-green)
              {% elif states('sensor.kitchen_notification').split('_')[1] == 'N'
              %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/kitchen
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:silverware-variant
        name: |
          {{states('sensor.dining_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.dining_notification').split('_')[1] == 'R' %}
                var(--notification-badge-red)
              {% elif states('sensor.dining_notification').split('_')[1] == 'G'
              %}
                var(--notification-badge-green)
              {% elif states('sensor.dining_notification').split('_')[1] == 'N'
              %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/dining
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:hail
        name: |
          {{states('sensor.guest_room_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.guest_room_notification').split('_')[1] ==
              'R' %}
                var(--notification-badge-red)
              {% elif states('sensor.guest_room_notification').split('_')[1] ==
              'G' %}
                var(--notification-badge-green)
              {% elif states('sensor.guest_room_notification').split('_')[1] ==
              'N' %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/guest-room
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:washing-machine
        name: |
          {{states('sensor.laundry_room_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 3px solid
          icon:
            '--mdc-icon-size': 30px
            transform: rotateY(180deg)
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.laundry_room_notification').split('_')[1] == 'R'
              %}
                var(--notification-badge-red)
              {% elif states('sensor.laundry_room_notification').split('_')[1] == 'G'
              %}
                var(--notification-badge-green)
              {% elif states('sensor.laundry_room_notification').split('_')[1] == 'N'
              %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/laundry-room
        hold_action:
          action: none
  - type: divider
    style:
      height: 0px
      background-color: var(--nav-color)
  - type: custom:paper-buttons-row
    buttons:
      - entity_id: script.dummy
        icon: mdi:fish
        name: |
          {{states('sensor.aquarium_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.aquarium_notification').split('_')[1] == 'R'
              %}
                var(--notification-badge-red)
              {% elif states('sensor.aquarium_notification').split('_')[1] ==
              'G' %}
                var(--notification-badge-green)
              {% elif states('sensor.aquarium_notification').split('_')[1] ==
              'N' %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/aquarium
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:television-speaker
        name: |
          {{states('sensor.entertainment_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.entertainment_notification').split('_')[1] ==
              'R' %}
                var(--notification-badge-red)
              {% elif states('sensor.entertainment_notification').split('_')[1]
              == 'G' %}
                var(--notification-badge-green)
              {% elif states('sensor.entertainment_notification').split('_')[1]
              == 'N' %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/entertainment
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:shield-home
        name: |
          {{states('sensor.security_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.security_notification').split('_')[1] == 'R'
              %}
                var(--notification-badge-red)
              {% elif states('sensor.security_notification').split('_')[1] ==
              'G' %}
                var(--notification-badge-green)
              {% elif states('sensor.security_notification').split('_')[1] ==
              'N' %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/security
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:heart-pulse
        name: |
          {{states('sensor.health_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.health_notification').split('_')[1] == 'R' %}
                var(--notification-badge-red)
              {% elif states('sensor.health_notification').split('_')[1] == 'G'
              %}
                var(--notification-badge-green)
              {% elif states('sensor.health_notification').split('_')[1] == 'N'
              %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/health
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:robot
        name: |
          {{states('sensor.tools_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.tools_notification').split('_')[1] == 'R' %}
                var(--notification-badge-red)
              {% elif states('sensor.tools_notification').split('_')[1] == 'G'
              %}
                var(--notification-badge-green)
              {% elif states('sensor.tools_notification').split('_')[1] == 'N'
              %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/tools
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:chart-bar
        name: |
          {{states('sensor.info_hub_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.info_hub_notification').split('_')[1] == 'R'
              %}
                var(--notification-badge-red)
              {% elif states('sensor.info_hub_notification').split('_')[1] ==
              'G' %}
                var(--notification-badge-green)
              {% elif states('sensor.info_hub_notification').split('_')[1] ==
              'N' %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/info-hub
        hold_action:
          action: none
      - entity_id: script.dummy
        icon: mdi:head-cog
        name: |
          {{states('sensor.system_info_notification').split('_')[0]}}
        layout: icon|name
        style:
          button:
            font-size: 12px
            padding: 7px 0px 7px 0px
            color: var(--nav-color)
            border-bottom: 0px solid
          icon:
            '--mdc-icon-size': 30px
          name:
            color: white
            font-weight: 1000
            margin-bottom: 10px
            margin-left: '-15px'
            background: >
              {% if states('sensor.system_info_notification').split('_')[1] ==
              'R' %}
                var(--notification-badge-red)
              {% elif states('sensor.system_info_notification').split('_')[1] ==
              'G' %}
                var(--notification-badge-green)
              {% elif states('sensor.system_info_notification').split('_')[1] ==
              'N' %}
                var(--notification-badge-neutral)
              {% else %} {% endif %}
            height: 20px
            width: 20px
            border-radius: 50%
            z-index: 1
            padding: 2px 2px 2px 3px
        tap_action:
          action: navigate
          navigation_path: /lovelace-home/system-info
        hold_action:
          action: none
```


  
________________________________________________________________________________________________________________________________________________________________________________________________
________________________________________________________________________________________________________________________________________________________________________________________________

# Using Google Generative AI Conversation Integration in HA to Analyze an Image
<image src="https://github.com/satyambaba/homeassistant/assets/6833101/797d8a43-3cf4-4d1c-96b5-caffec4a5b11" width="400" />

**Note: This integration works with any image stored locally including camera snapshots.**

# Steps to follow

**1. Add and configure [Google Generative AI Conversation Integration](https://www.home-assistant.io/integrations/google_generative_ai_conversation?fbclid=IwAR3MOTjh6NoxIa4i4NVu5s23xE_YAnc_nJ7ign8iX4ZzRAg1dAEToZfanC8) in HA**

**2. Create a Text helper called "Doorbell AI Description" (Settings > Devices & services > Helpers > Create Helper > Text)**

**3. Create a folder "/config/www/picture/doorbell"**

**4. Put below code in the config file and reboot (it is only needed when you are using Step 5, Method 1)**
```yaml
downloader:
  download_dir: /config/www/picture/doorbell
```

**5. Script to save camera image to a local location (you can use either of the methods below based on the camera you have)**

**Method 1**
```yaml
doorbell_snapshot:
  alias: Doorbell Snapshot
  sequence:
    - service: downloader.download_file
      data:
        url: >-
          http://[enter your HA IP]:8123{{state_attr('image.doorbell_event_image','entity_picture')}}
        filename: visitor.jpg
        overwrite: true
```

**Method 2**
```yaml
doorbell_snapshot2:
  alias: Doorbell Snapshot 2
  sequence:
    - service: camera.snapshot
      target:
        entity_id: camera.doorbell
      data:
        filename: /config/www/picture/doorbell/visitor.jpg
```

**6. Script to generate description using Google Generative AI**
```yaml
doorbell_snapshot_ai_description:
  alias: Doorbell Snapshot AI Description
  sequence:
    - service: google_generative_ai_conversation.generate_content
      data:
        image_filename: /config/www/picture/doorbell/visitor.jpg
        prompt: >-
          Very briefly describe, what do you see in this image from my doorbell camera. Your message needs to be short to be fit in a phone notification. Don't describe stationary objects or buildings.
      response_variable: doorbell_ai_description
    - service: input_text.set_value
      data:
        value: >
          {{doorbell_ai_description['text']}}
          # in case this doesn't work, you may try {{doorbell_ai_description.text}}
      target:
        entity_id: input_text.doorbell_ai_description
```

**7. Leverage input_text.doorbell_ai_description entity in the automation for notification or anywhere else you want**

________________________________________________________________________________________________________________________________________________________________________________________________
________________________________________________________________________________________________________________________________________________________________________________________________

# Fitness Stats Comparison Card
A card to compare fitness stats using HA Fitbit or other integrations. In this case, I used the info provided by the official HA Fitbit integration and a custom Fitbit integration because the official HA Fitbit integration doesn't support multiple accounts. If other fitness integrations provide similar information, sensors can be modified easily.

![image](https://github.com/satyambaba/homeassistant/assets/6833101/40a6c69f-21e9-4ab9-bd4d-5d8d1820203f)

**1. Features**
- This card displays and compares key fitness stats in a concise and organized manner to keep us motivated
- Stars are earned by comparing steps, active hours and sleep hours
- Winner is decided based on the number of stars earned
- Individual icons are colored based on sensor values. For example - Step icons are green if steps are greater than 10K, orange if steps are between 3K - 10K and red if steps are less than 3K
- Even if you only have the official HA Fitbit integration, this card can be easily modified to display the stats only for one person

**2. Steps to use**
- Install cards mentioned in 1.4
- Create template sensors mentioned in 1.5
- Use YAML code mentioned in 1.3
  
**3. YAML Code for the Card**
```yaml
type: custom:vertical-stack-in-card
style: |
  ha-card{
    background: rgba(25, 25, 25, 0.3);
    padding: 15px 0px 15px 0px;
  }
cards:
  - type: custom:stack-in-card
    style: |
      ha-card{
        background: rgba(25, 25, 25, 0);
      }
    mode: horizontal
    cards:
      - type: custom:vertical-stack-in-card
        style: |
          ha-card{
            background: rgba(25, 25, 25, 0);
            border-radius: 0px;
            border-right: 1px solid #787878;
          }
        cards:
          - type: custom:vertical-stack-in-card
            style: |
              ha-card {
                background: none;
                box-shadow: none;
              }
            cards:
              - type: custom:paper-buttons-row
                styles:
                  padding: 0px 0px 15px 0px
                buttons:
                  - entity: person.person1
                    name: >
                      {% if states('sensor.person1_fitness_score') >
                      states('sensor.person2_fitness_score') %}Person 1 🏆{% else
                      %}Person 1{% endif %}
                    layout: icon_name
                    image: >
                        {{state_attr('person.person1','entity_picture')}}
                    style:
                      button:
                        font-size: 1.1rem
                        font-weight: 500
                        padding: 0px 0px 0px 0px
                      icon:
                        height: 50px
                        width: 50px
                      name:
                        padding: 5px 0px 0px 0px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 10px 0px 10px 0px
                buttons:
                  - entity: sensor.person1_distance
                    icon: mdi:map-marker-distance
                    name: |
                      {{states(config.entity) | round(1)}} km
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                        transform: rotateY(0deg)
                    tap_action:
                      action: none
                    hold_action:
                      action: none
                  - entity: sensor.person1_steps
                    icon: mdi:shoe-sneaker
                    name: >
                      {{"{:,.0f}".format(states(config.entity) | int)}} {% if
                      (states('sensor.person1_steps') | float) >
                      (states('sensor.person2_steps') | float) %}⭐{% else %}{% endif %}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                        transform: rotateY(180deg)
                        color: >
                          {% if states(config.entity) | float(0) < 3000
                          %}var(--threshold-red) {% elif states(config.entity) |
                          float(0) < 10000 %}var(--threshold-orange) {% else
                          %}var(--threshold-green){% endif %}
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 10px 0px 10px 0px
                buttons:
                  - entity: sensor.person1_inactive_hours
                    icon: mdi:seat-recline-normal
                    name: >
                      {% if states(config.entity) | float(0) < 1 %}
                      {{((states(config.entity)) | float * 60) | round(0)}} m {%
                      else %} {{(states(config.entity) | float(0)) | round(1)}}
                      h {% endif %}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
                  - entity: sensor.person1_active_hours
                    icon: mdi:run-fast
                    name: >
                      {% if states(config.entity) | float(0) < 1 %}
                      {{((states(config.entity)) | float * 60) | round(0)}} m {%
                      else %} {{(states(config.entity) | float(0)) | round(1)}}
                      h {% endif %} {% if (states('sensor.person1_active_hours') |
                      float) > (states('sensor.person2_active_hours') | float)
                      %}⭐{% else %}{% endif %}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 10px 0px 10px 0px
                buttons:
                  - entity: sensor.person1_sleep_efficiency
                    icon: >
                      {% if states(config.entity) | float(0) >= 60
                      %}mdi:thumb-up {% else %}mdi:thumb-down{% endif %}
                    name: |
                      {{states(config.entity)}}%
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 22px
                        transform: >
                          {% if states(config.entity) | float(0) >= 60
                          %}rotateY(0deg){% else %}rotateY(180deg){% endif %}
                        color: >
                          {% if states(config.entity) | float(0) >= 60
                          %}var(--threshold-green) {% else
                          %}var(--threshold-red){% endif %}
                    tap_action:
                      action: none
                    hold_action:
                      action: none
                  - entity: sensor.person1_sleep_hours
                    icon: mdi:bed-clock
                    name: >
                      {{(states(config.entity) | float(0)) | round(1)}} h {% if
                      (states('sensor.person1_sleep_hours') | float) >
                      (states('sensor.person2_sleep_hours') | float) %}⭐{% else
                      %}{% endif %}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 22px
                        transform: rotateY(180deg)
                        color: >
                          {% if states(config.entity) | float(0) >= 7
                          %}var(--threshold-green) {% else
                          %}var(--threshold-red){% endif %}
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 10px 0px 10px 0px
                buttons:
                  - entity: sensor.versa_3_battery
                    icon: >
                      {% if states(config.entity) == 'Low' %}mdi:battery-30 {%
                      elif states(config.entity) == 'Medium' %}mdi:battery-60{%
                      elif states(config.entity) == 'High' %}mdi:battery-90{%
                      else %}mdi:battery-60{% endif %}
                    name: |
                      {{states(config.entity)[0]}}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 22px
                        color: >
                          {% if states(config.entity) == 'Low'
                          %}var(--threshold-red){% elif states(config.entity) ==
                          'Medium' %}var(--threshold-orange){% elif
                          states(config.entity) == 'High'
                          %}var(--threshold-green){% else %}{% endif %}
                      name:
                        padding: 7px 0px 0px 0px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
                  - entity: sensor.person1_resting_heart_rate
                    icon: mdi:heart-pulse
                    name: |
                      {{states(config.entity) | replace('unknown', '.')}}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                        transform: rotateY(180deg)
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 15px 0px 0px 0px
                buttons:
                  - entity: sensor.person1_scoreboard_update_time
                    name: |
                      Updated {{states(config.entity)}} ago
                    layout: name
                    style:
                      button:
                        font-size: 0.9rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 22px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
      - type: custom:vertical-stack-in-card
        style: |
          ha-card{
            background: rgba(25, 25, 25, 0);
            border-radius: 0px;
          }
        cards:
          - type: custom:vertical-stack-in-card
            style: |
              ha-card {
                background: none;
                box-shadow: none;
              }
            cards:
              - type: custom:paper-buttons-row
                styles:
                  padding: 0px 0px 15px 0px
                buttons:
                  - entity: person.person2
                    name: >
                      {% if states('sensor.person1_fitness_score') <
                      states('sensor.person2_fitness_score') %}Person 2 🏆{% else
                      %}Person 2{% endif %}
                    layout: icon_name
                    image: >
                        {{state_attr('person.person2','entity_picture')}}
                    style:
                      button:
                        font-size: 1.1rem
                        font-weight: 500
                        padding: 0px 0px 0px 0px
                      icon:
                        height: 50px
                        width: 50px
                      name:
                        padding: 5px 0px 0px 0px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 10px 0px 10px 0px
                buttons:
                  - entity: sensor.person2_steps
                    icon: mdi:shoe-sneaker
                    name: >
                      {% if (states('sensor.person1_steps') | float) <
                      (states('sensor.person2_steps') | float) %}⭐{% else %}{% endif %}
                      {{"{:,.0f}".format(states(config.entity) | int)}}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                        color: >
                          {% if states(config.entity) | float(0) < 3000
                          %}var(--threshold-red) {% elif states(config.entity) |
                          float(0) < 10000 %}var(--threshold-orange) {% else
                          %}var(--threshold-green){% endif %}
                    tap_action:
                      action: none
                    hold_action:
                      action: none
                  - entity: sensor.person2_distance
                    icon: mdi:map-marker-distance
                    name: |
                      {{states(config.entity) | round(1)}} km
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                        transform: rotateY(180deg)
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 10px 0px 10px 0px
                buttons:
                  - entity: sensor.person2_active_hours
                    icon: mdi:run-fast
                    name: >
                      {% if (states('sensor.person1_active_hours') | float) <
                      (states('sensor.person2_active_hours') | float) %}⭐{% else
                      %}{% endif %} {% if states(config.entity) | float(0) < 1
                      %} {{((states(config.entity)) | float * 60) | round(0)}} m
                      {% else %} {{(states(config.entity) | float(0) ) |
                      round(1)}} h {% endif %}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                        transform: rotateY(180deg)
                    tap_action:
                      action: none
                    hold_action:
                      action: none
                  - entity: sensor.person2_inactive_hours
                    icon: mdi:seat-recline-normal
                    name: >
                      {% if states(config.entity) | float(0) < 1 %}
                      {{((states(config.entity)) | float * 60) | round(0)}} m {%
                      else %} {{(states(config.entity) | float(0)) | round(1)}}
                      h {% endif %}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                        transform: rotateY(180deg)
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 10px 0px 10px 0px
                buttons:
                  - entity: sensor.person2_sleep_hours
                    icon: mdi:bed-clock
                    name: >
                      {% if (states('sensor.person1_sleep_hours') | float) <
                      (states('sensor.person2_sleep_hours') | float) %}⭐{% else
                      %}{% endif %} {{(states(config.entity) | float(0)) |
                      round(1)}} h
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 22px
                        transform: rotateY(0deg)
                        color: >
                          {% if states(config.entity) | float(0) >= 7
                          %}var(--threshold-green) {% else
                          %}var(--threshold-red){% endif %}
                    tap_action:
                      action: none
                    hold_action:
                      action: none
                  - entity: sensor.person2_sleep_efficiency
                    icon: >
                      {% if states(config.entity) | float(0) >= 60
                      %}mdi:thumb-up {% else %}mdi:thumb-down{% endif %}
                    name: |
                      {{states(config.entity)}}%
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 22px
                        transform: >
                          {% if states(config.entity) | float(0) >= 60
                          %}rotateY(180deg){% else %}rotateY(0deg){% endif %}
                        color: >
                          {% if states(config.entity) | float(0) >= 60
                          %}var(--threshold-green) {% else
                          %}var(--threshold-red){% endif %}
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 10px 0px 10px 0px
                buttons:
                  - entity: sensor.person2_resting_heart_rate
                    icon: mdi:heart-pulse
                    name: |
                      {{states(config.entity) | replace('unknown', '.')}}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 27px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
                  - entity: sensor.charge_5_battery
                    icon: >
                      {% if states(config.entity) == 'Low' %}mdi:battery-30 {%
                      elif states(config.entity) == 'Medium' %}mdi:battery-60{%
                      elif states(config.entity) == 'High' %}mdi:battery-90{%
                      else %}mdi:battery-60{% endif %}
                    name: |
                      {{states(config.entity)[0]}}
                    layout: icon_name
                    style:
                      button:
                        font-size: 1rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 22px
                        color: >
                          {% if states(config.entity) == 'Low'
                          %}var(--threshold-red){% elif states(config.entity) ==
                          'Medium' %}var(--threshold-orange){% elif
                          states(config.entity) == 'High'
                          %}var(--threshold-green){% else %}{% endif %}
                      name:
                        padding: 7px 0px 0px 0px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
              - type: custom:paper-buttons-row
                styles:
                  padding: 15px 0px 0px 0px
                buttons:
                  - entity: sensor.person2_scoreboard_update_time
                    name: |
                      Updated {{states(config.entity)}} ago
                    layout: name
                    style:
                      button:
                        font-size: 0.9rem
                        padding: 0px 0px 0px 0px
                      icon:
                        '--mdc-icon-size': 22px
                    tap_action:
                      action: none
                    hold_action:
                      action: none
```
**4. Requirements**
- [Vertical Stack In Card](https://github.com/ofekashery/vertical-stack-in-card)
- [Paper Buttons Row Card](https://github.com/jcwillox/lovelace-paper-buttons-row)
- [Card Mod](https://github.com/thomasloven/lovelace-card-mod)

**5. Details about Sensors**

- Sensors Provided by the official HA Fitbit Integration which are used (directly or in a template sensor) for this card - Steps, Distance, Minutes Fairly Active, Minutes Lightly Active, Minutes Very Active, Minutes Sedentary, Sleep Efficiency, Minutes Asleep, Resting Heart Rate and Battery
  
- Sleep Hours - Template Sensor to calculate sleep hours
  
  ```yaml
    person1_sleep_hours:
      friendly_name: "Person1 Sleep Hours"
      value_template: >
        {{(states('sensor.person1_sleep_minutes_asleep') | float(0) / 60) | round(1)}}
  ```
  Please create the same sensor for person2.

- Active Hours - Template Sensor to calculate active hours
  
  ```yaml
    person1_active_hours:
      friendly_name: "Person1 Active Hours"
      value_template: >
        {{((states('sensor.person1_minutes_lightly_active') | float(0) + states('sensor.person1_minutes_fairly_active') | float(0) + states('sensor.person1_minutes_very_active') | float(0)) / 60) | round(1)}}
  ```
  Please create the same sensor for person2.

- Inactive Hours - Template Sensor to calculate inactive hours
  
  ```yaml
    person1_inactive_hours:
      friendly_name: "Person1 Inactive Hours"
      value_template: >
        {{(states('sensor.person1_minutes_sedentary') | float(0) / 60) | round(1)}}
  ```
  Please create the same sensor for person2.

- Active Hours Percentage - Template Sensor to calculate active hours percentage
  
  ```yaml
    person1_active_hours_percentage:
      friendly_name: "Person1 Active Hours Percentage"
      device_class: "battery"
      unit_of_measurement: "%"
      value_template: >
        {{(states('sensor.person1_active_hours') | float * 100 / (states('sensor.person1_active_hours') | float + states('sensor.person1_inactive_hours') | float)) | round(0) }}
  ```
  Please create the same sensor for person2.

- Fitness Wins - Template Sensor to store "wins" info based on comparing steps, active hours and sleep hours

  ```yaml
    person1_fitness_wins:
      friendly_name: "Person1 Fitness Wins"
      value_template: >
        {% if (states('sensor.person1_steps') | float) >
        (states('sensor.person2_steps') | float) %}1{% else %}0{% endif %}{%
        if (states('sensor.person1_active_hours') | float) >
        (states('sensor.person2_active_hours') | float) %}1{% else %}0{% endif %}{% if
        (states('sensor.person1_sleep_hours') | float) >
        (states('sensor.person2_sleep_hours') | float) %}1{% else %}0{% endif %}
    ```
    Please create the same sensor for person2.

- Fitness Score - Template Sensor to calculate score based on wins.
  
  ```yaml
      person1_fitness_score:
      friendly_name: "Person1 Fitness Score"
      value_template: >
        {{((states('sensor.person1_fitness_wins')[0] | int) + (states('sensor.person1_fitness_wins')[1] | int) + (states('sensor.person1_fitness_wins')[2] | int)) | int}}
  ```
  Please create the same sensor for person2.

  - Scoreboard Update Time - Template Sensor to calculate the scoreboard last update time.
  
  ```yaml
    person1_scoreboard_update_time:
      friendly_name: Person1 Scoreboard Update Time
      value_template: >
        {% if ((as_timestamp(now())-max(as_timestamp(states.sensor.person1_steps.last_updated),as_timestamp(states.sensor.person1_resting_heart_rate.last_updated),as_timestamp(states.sensor.person1_sleep_minutes_asleep.last_updated),as_timestamp(states.sensor.versa_3_battery.last_updated)))/3600) | float > 1 %}{{(((as_timestamp(now())-max(as_timestamp(states.sensor.person1_steps.last_updated),as_timestamp(states.sensor.person1_resting_heart_rate.last_updated),as_timestamp(states.sensor.person1_sleep_minutes_asleep.last_updated),as_timestamp(states.sensor.versa_3_battery.last_updated)))/3600) | int)}}h
        {% else %}{% endif %}{{(((((as_timestamp(now())-max(as_timestamp(states.person1_fitbit_steps.last_updated),as_timestamp(states.sensor.person1_resting_heart_rate.last_updated),as_timestamp(states.sensor.person1_sleep_minutes_asleep.last_updated),as_timestamp(states.sensor.versa_3_battery.last_updated)))/3600) | float)-(((as_timestamp(now())-max(as_timestamp(states.sensor.person1_steps.last_updated),as_timestamp(states.sensor.person1_resting_heart_rate.last_updated),as_timestamp(states.sensor.person1_sleep_minutes_asleep.last_updated),as_timestamp(states.sensor.versa_3_battery.last_updated)))/3600) | int))*60) | round(0)}}m

  ```
  Please create the same sensor for person2.
