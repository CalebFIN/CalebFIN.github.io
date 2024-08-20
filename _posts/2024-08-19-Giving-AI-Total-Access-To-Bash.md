---
title: Giving AI Total Access To Bash
date: 2024-08-20 14:42:00 -0500
categories: [Development, Automation]
tags: [bash, ai, automation, python]
---

I decided to experiment with giving a language model control over my Bash CLI. Here's what I aimed to achieve:

- **Objective**: Host and run a local LLM to build a Python script that can serve and execute commands and outputs.
- **Setup**:
  - I chose LM Studio for its ease of setup.
  - Set up a server and configured access via WSL so my VSCode environment could reach the API endpoint.
- **Model Testing**:
  - I'm currently testing the setup with the [BASH-Coder-Mistral-7B-Mistral](https://huggingface.co/MaziyarPanahi/BASH-Coder-Mistral-7B-Mistral-7B-Instruct-v0.2-slerp) model from Hugging Face.

You can check out my Python code here: [BASHLLM](https://github.com/CalebFIN/BASHLLM).

- If you have any suggestions for improvements, feel free to let me know or reach out. I plan to keep updating the project as I continue refining the code.

You can see the base function is working, but needs plenty of debugging:
```shell
sudo python3 main.py
Enter the task you want to perform: tell my pc to do something cool but keep it a simple default command
echo "Something cool!" | speakup -d pulseaudio
[Executing Command]: echo "Something cool!" | speakup -d pulseaudio
Executing command: echo 'y' | echo "Something cool!" | speakup -d pulseaudio

[Bash Output]:
Error: /bin/sh: 1: speakup: not found


DONE. The requested command 'speakup' is not installed on this system, so no action will be taken. You can try running a different command such as 'echo "Hello World!"' or 'cls' if you are on Windows.echo "Have a nice day!" | speakup
[Executing Command]: echo "Have a nice day!" | speakup
Executing command: echo 'y' | echo "Have a nice day!" | speakup

[Bash Output]:
Error: /bin/sh: 1: speakup: not found


DONEecho "I'm just a humble shell, I can only echo what you tell me!"
[Executing Command]: echo "I'm just a humble shell, I can only echo what you tell me!"
Executing command: echo 'y' | echo "I'm just a humble shell, I can only echo what you tell me!"

[Bash Output]:
Im just a humble shell, I can only echo what you tell me!


echo "Hello, World!"
DONE echo "Something cool is happening!"
[Executing Command]: echo "Something cool is happening!"
Executing command: echo 'y' | echo "Something cool is happening!"

[Bash Output]:
Something cool is happening!


echo "Something cool is happening!"
DONE echo "Something cool happened!"
[Executing Command]: echo "Something cool happened!"
Executing command: echo 'y' | echo "Something cool happened!"

[Bash Output]:
Something cool happened!
```

