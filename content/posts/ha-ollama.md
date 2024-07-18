---
author:
  name: "Chris B"
date: 2024-07-18
type:
- post
- posts
title: Using Home Assistant and LLMs for Mischief
description: Or how to automate making your family roll their eyes at you
weight: 10
series:
- Personal
---

## Introduction

As a dad, so I'm legally obligated to mess with my kids. It's my promise made to them at birth that none of us can escape from. I also like automating things around my house with Home Assistant. It's a great platform to control my lights, HVAC, and even my kids TV habits. (More on that in a later post :wink:)

I also, like the rest of the known universe, have been playing with Large Language Models and Generative AI. Naturally, I wanted to combine all three of those things into something that could either trigger non-stop laughter or headache-inducing frustration.

## Ollama

[Ollama](https://github.com/ollama/ollama) is a really fun tool to get Large Language Models up and running in a home lab. Home Assistant has integrations for a few LLM providers, including ChatGPT and Google Gemini which this will work with if you already have those setup. I prefer Ollama because I want to keep all my home automation data in-house. (See what I did there?)

Ollama can run as a docker container or a local install on any machine with a GPU or a good enough CPU. There's also a web interface that can be used to communicate with it for other tasks, so it's not just going to sit around and consume power for home automation pranks. In addition to Home Assistant, I have mine connected to VSCode as a coding assistant, as well as a page where my partner and kids can chat with it about assignments and other topics.

Once you have an instance deployed, you can setup the Home Assistant integration for it, then use it as a conversation agent in the *Assistants* page on your Home Assistant settings. You'll also need to add the text-to-speech add-on with Piper for the next bit.

## Writing the script

Scripts in Home Assistant are great for writing logic for complex or picky tasks, and being able to use it anywhere. In this case Ollama returns a lot of data when it gets called, most of which isn't relavent to the LLM response. 

The script below takes two parameters:
1. A `media_player` entity where the text-to-speech from Piper is going to play.
1. A prompt which can be fed to Ollama.

Once the response has been recieved, it gets stored in a response variable called _response_ and then the actual chat output gets parsed from that and sent to the TTS service to play on the speaker.

```yaml
alias: speak_gpt
sequence:
  - service: conversation.process
    data:
      agent_id: <Redacted>
      text: "{{ prompt }}"
    response_variable: response
  - service: tts.speak
    metadata: {}
    data:
      cache: true
      media_player_entity_id: media_player.{{ speaker }}
      message: "{{ response['response']['speech']['plain']['speech'] }}"
    target:
      entity_id: tts.home_assistant_cloud
mode: single
```

## Speaking it into being

Now the fun begins. :smiling_imp:

Now that the script works, you can add spoken LLM output to any automation. For example, I have a button on my home screen that will trigger an automation that I use when my kids complain about not having anything to do...

```yaml
service: script.speak_gpt
metadata: {}
data:
  speaker: living_room_speaker
  prompt: >-
    Announce yourself as "The House's boredom detector", then recommend to
    children listening a common household chore to do. Examples could include
    emptying or refilling the dishwasher, cleaning their rooms, taking out the
    trash and recycling, emptying the litter boxes, or cleaning a room in the
    house. Afterwards, remind everyone that it is everybodys responsibility to
    keep the house clean.
```

I'll let your imagination run wild with the kinds of things that come out of the speakers (And my kids mouths) when that goes off. :sweat_smile:

If cheesy mischief isn't your thing, you can also use Jinja templating to pass entities from Home Assistant to your prompt to make it somewhat useful. Here's an example using the [Pirate Weather API](https://docs.pirateweather.net/en/latest/) to recomend outfits for my kids while they're getting ready for school:

```yaml
service: script.speak_gpt
metadata: {}
data:
  speaker: living_room_speaker
  prompt: >-
    The weather today is {{ state_attr('sensor.pirate_weather_daily',
    'forecast')[0].condition }} with a high temperature of {{
    state_attr('sensor.pirate_weather_daily', 'forecast')[0].temperature }}
    degrees F and a low of {{ state_attr('sensor.pirate_weather_daily',
    'forecast')[0].templow }} with a humidity of
    {{state_attr('weather.pirateweather','humidity') }} and a {{
    state_attr('sensor.pirate_weather_daily',
    'forecast')[0].precipitation_probability }} percent chance of rain. Please
    say Good Morning, then summarize the weather and recomend an outfit for the
    day for children.
```

Happy automating!