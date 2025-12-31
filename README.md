# Twilio SMS for Home Assistant

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/hacs/integration)

A Home Assistant custom integration to send SMS and MMS messages via Twilio.

## Features

- Send SMS messages to multiple recipients
- Send MMS with media URLs (images, etc.) - Twilio fetches from the URL
- Local file support (auto-converts `/local/` paths to external URLs)
- Jinja2 template support for dynamic messages
- Multiple Twilio phone numbers support
- Easy UI-based configuration

## Installation

### HACS (Recommended)

1. Open HACS in Home Assistant
2. Click on "Integrations"
3. Click the three dots menu and select "Custom repositories"
4. Add this repository URL and select "Integration" as the category
5. Click "Add"
6. Search for "Twilio SMS" and install it
7. Restart Home Assistant

### Manual Installation

1. Download the `custom_components/twilio_sms` folder
2. Copy it to your Home Assistant `config/custom_components/` directory
3. Restart Home Assistant

## Configuration

1. Go to **Settings > Devices & Services**
2. Click **Add Integration**
3. Search for "Twilio SMS"
4. Enter your Twilio Account SID and Auth Token
   - Find your credentials at:
     - [US Console](https://console.twilio.com/us1/account/keys-credentials/api-keys)
     - [Australia Console](https://console.twilio.com/au1/account/keys-credentials/api-keys)
     - [Ireland Console](https://console.twilio.com/ie1/account/keys-credentials/api-keys)
5. Select the phone numbers you want to use

## Usage

### Service: `twilio_sms.send_message`

Send an SMS or MMS message.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `target` | Yes | Phone number(s) to send to (E.164 format). Supports Jinja2 templates. |
| `message` | Yes | Message body. Supports Jinja2 templates. |
| `from_number` | Yes | Your Twilio phone number to send from (dropdown). |
| `media_url` | No | URL(s) for MMS media. Supports `/local/` paths (auto-converted to URLs) and Jinja2 templates. |

### Examples

#### Basic SMS

```yaml
service: twilio_sms.send_message
data:
  target: "+15551234567"
  message: "Hello from Home Assistant!"
  from_number: "+15559876543"
```

#### SMS to Multiple Recipients

```yaml
service: twilio_sms.send_message
data:
  target:
    - "+15551234567"
    - "+15551234568"
  message: "Alert! Motion detected."
  from_number: "+15559876543"
```

#### Using Jinja2 Templates

```yaml
service: twilio_sms.send_message
data:
  target: "{{ states('input_text.my_phone') }}"
  message: |
    Home Status Report:
    Temperature: {{ states('sensor.temperature') }}°F
    Humidity: {{ states('sensor.humidity') }}%
    Time: {{ now().strftime('%H:%M') }}
  from_number: "+15559876543"
```

#### MMS with External Image

```yaml
service: twilio_sms.send_message
data:
  target: "+15551234567"
  message: "Check out this image!"
  media_url: "https://example.com/image.jpg"
  from_number: "+15559876543"
```

#### MMS with Local Camera Snapshot

```yaml
# First, save snapshot to /config/www/ folder
- service: camera.snapshot
  target:
    entity_id: camera.front_door
  data:
    filename: /config/www/snapshot.jpg

# Then send via MMS using /local/ path (auto-converted to external URL)
- service: twilio_sms.send_message
  data:
    target: "+15551234567"
    message: "Motion detected at front door!"
    media_url: "/local/snapshot.jpg"
    from_number: "+15559876543"
```

### Media URL Notes

Twilio fetches media from URLs, so they must be publicly accessible.

**External URLs:** Use any public URL directly.

**Local files:**
- Place files in `/config/www/` folder
- Reference them as `/local/filename.jpg`
- The integration auto-converts to your external HA URL
- **Requirement:** Home Assistant must be externally accessible (Nabu Casa, reverse proxy, or port forwarding)

Supported local paths: `/local/`, `/media/`, `/api/`

## Automation Example

```yaml
automation:
  - alias: "Send SMS on Motion"
    trigger:
      - platform: state
        entity_id: binary_sensor.front_door_motion
        to: "on"
    action:
      # Save snapshot to /config/www/ folder
      - service: camera.snapshot
        target:
          entity_id: camera.front_door
        data:
          filename: /config/www/motion.jpg
      - delay: "00:00:02"
      # /local/motion.jpg is auto-converted to https://your-ha-url/local/motion.jpg
      - service: twilio_sms.send_message
        data:
          target: "+15551234567"
          message: "Motion detected at {{ now().strftime('%H:%M:%S') }}"
          media_url: "/local/motion.jpg"
          from_number: "+15559876543"
```

## Troubleshooting

### Media not sending
- Ensure Home Assistant is accessible from the internet
- Check that the file exists in `/config/www/`
- Verify your external URL is configured in Home Assistant settings

### Invalid credentials
- Double-check your Account SID and Auth Token from Twilio Console
- Ensure you're using the main account credentials, not API keys

### Phone number not found
- Verify the phone number is active in your Twilio account
- Check that messaging is enabled for the number

## License

MIT License
