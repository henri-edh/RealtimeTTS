# RealtimeTTS FAQ

Welcome to the RealtimeTTS frequently answered questions and troubleshooting guide! Below is a comprehensive list of common issues and their solutions.

## Table of Contents

1. [Stream to Web Browser](#stream-to-web-browser)
2. [Voice Cloning](#use-voice-cloning)
3. [Pretrained XTTS Model](#use-a-pretrained-xtts-model)
4. [Stuttering](#stuttering-what-to-do)
5. [UI Showcase Video](#use-an-ui-like-in-the-showcase-video)
6. [Use with LLM](#use-it-with-a-llm)
7. [RuntimeError "freeze_support"](#i-get-runtimeerror-freeze_support)
8. [Save Audio to File](#save-the-audio-to-file)
9. [Issues on Linux/Unix](#i-have-probs-running-this-on-linuxunix)
10. [Change Output Device](#i-want-to-use-another-output-device)
11. [Realtime Generated Audiochunks](#work-with-the-realtime-generated-audiochunks)
12. [Use RealtimeTTS in Another Language](#use-realtimetts-in-another-language)
13. [How to use Voices](#how-to-use-voices)
14. [Script won't Terminate](#script-wont-terminate)

## Stream to Web Browser

If you're looking to stream audio to a web browser, check out the [FastAPI example](https://github.com/KoljaB/RealtimeTTS/tree/master/example_fast_api).

## Use Voice Cloning

To clone a voice, prepare a wave file containing a short (~10-30 sec) voice sample in 22050 Hz mono int 16bit. Submit the filename as the "voice" parameter to the CoquiEngine constructor.

## Use a Pretrained XTTS Model

To use a custom pretrained XTTS model:
- Submit the name of the directory containing the model files for the custom model as the "specific_model" parameter to the CoquiEngine constructor.
- Specify the directory path in the "local_models_path" parameter of the CoquiEngine constructor.

## Stuttering - What to Do

If stuttering occurs:
- For ElevenlabsEngine, ensure you're using a model specified for realtime usage, such as v1 models.
- For CoquiEngine:
  1. Ensure pyTorch is installed with CUDA support.
    `pip install torch==2.2.2+cu118 torchaudio==2.2.2 --index-url https://download.pytorch.org/whl/cu118
` (adjust 2.2.2 to your desired torch version and 118 to your desired CUDA version)
  2. If the system is too slow for realtime synthesis, set "full_sentences=True" as a parameter for the CoquiEngine constructor which prevents mid-sentence stuttering.

## Use an UI like in the Showcase Video

Check out the code used for the video on the front page of the repository [here](https://github.com/KoljaB/RealtimeTTS/blob/master/tests/pyqt6_speed_test.py).

## Use it with a LLM

Feed a creation stream:
```python
    stream = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": "A three-sentence relaxing speech."}],
        stream=True,
    )

    TextToAudioStream(CoquiEngine(), log_characters=True).feed(stream).play()
```

Or create your own generator and feed that:
```python
    def generator():
        for chunk in llm.return_stream:

            # do something with the chunks (filter out stuff etc)
            content = chunk.get("content") # in case chunks aren't strings

            if some_condition_with_the_chunks:
                yield content # that should be string

    stream.feed(generator())
```

## I get RuntimeError "freeze_support"

Add entry protection to your code.

Write
```python
if __name__ == '__main__':
```
and put your code behind that.

The lib uses multiprocessing, so `if __name__ == '__main__':` is needed to prevent unexpected behavior - for a detailed explanation pls look at the [official Python documentation on `multiprocessing`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing-programming).

## Save the audio to file

The stream.play and stream.play_async methods have a parameter "output_wavfile" where you can specify the file the synthesis should be written into.

[Here](https://github.com/KoljaB/RealtimeTTS/blob/master/tests/write_to_file.py) is an example showcasing this.

You may want to use this together with the parameter "muted=True" of stream.play and stream.play_async methods, that prevents that the synthesis written to the files gets streamed to the output device.

## I have probs running this on Linux/Unix

Try:

```bash
sudo apt install portaudio19-dev
```

If this is not sufficient you might also try:
```bash
sudo apt update
sudo apt install python3-dev build-essential
sudo apt install gcc
sudo apt install libportaudio2 python3-pyaudio
sudo apt install espeak ffmpeg libespeak1
sudo apt install libasound-dev
```

## I want to use another output device

Specify your desired output as output_device_index parameter to the constructor of TextToAudioStream.

## Work with the realtime generated audiochunks

If you want to process the chunks generated by RealtimeTTS you can use the "on_audio_chunk" callback parameter of stream.play and stream.play_async methods, which submits the raw pcm chunk data as a parameter. You might want to retrieve the exact structure of these chunks using the get_stream_info-method of the engine you are using. This method returns a tuple containing the audio format, number of channels and sample rate of the chunks.

Example for Azure engine:

```python
    def get_stream_info(self):
        """
        Returns the PyAudio stream configuration information suitable for Azure Engine.

        Returns:
            tuple: A tuple containing the audio format, number of channels, and the sample rate.
                  - Format (int): The format of the audio stream. pyaudio.paInt16 represents 16-bit integers.
                  - Channels (int): The number of audio channels. 1 represents mono audio.
                  - Sample Rate (int): The sample rate of the audio in Hz. 16000 represents 16kHz sample rate.
        """
        return pyaudio.paInt16, 1, 16000
```

## Use RealtimeTTS in another language

### Setting the language

- For OpenAIEngine, feed the text in your desired language.
- For SystemEngine and AzureEngine, select a voice that supports your desired language.
- For ElevenlabsEngine, use the eleven_multilingual_v1 model.
- For CoquiEngine, specify the shortcut of your desired language as the "language" parameter to the constructor.

```python
CoquiEngine(language = "zh")
```

CoquiEngine with XTTS v2.0.2 supports 16 languages: English (en), Spanish (es), French (fr), German (de), Italian (it), Portuguese (pt), Polish (pl), Turkish (tr), Russian (ru), Dutch (nl), Czech (cs), Arabic (ar), Chinese (zh-cn), Japanese (ja), Hungarian (hu) and Korean (ko). v2.0.3 model also supports Hindi (hi).

### Use of Tokenizers

Tokenizers play a crucial role in the real-time processing and delivery of text-to-speech (TTS) synthesis in the RealtimeTTS library. They are responsible for breaking down the incoming text stream into individual sentences, which can then be processed and synthesized more efficiently.

#### NLTK Tokenizer

NLTK (Natural Language Toolkit) is a popular Python library for natural language processing. In the RealtimeTTS library, NLTK is used as one of the tokenizers available for sentence splitting.

**Usage**:

When initializing the TextToAudioStream object, you can specify the NLTK tokenizer by setting the `tokenizer` parameter to `"nltk"`. This tokenizer is suitable for basic sentence splitting tasks and is often used for its simplicity and ease of integration.

#### Stanza Tokenizer

Stanza (formerly known as StanfordNLP) is another powerful natural language processing library that provides state-of-the-art pretrained models for various NLP tasks, including tokenization and sentence splitting.

**Usage**:

To utilize the Stanza tokenizer, set the `tokenizer` parameter to `"stanza"` during the initialization of the TextToAudioStream object. Stanza offers more advanced tokenization capabilities compared to NLTK, including support for multiple languages and improved accuracy in detecting sentence boundaries.

#### Choosing the Right Tokenizer

When selecting a tokenizer for your application, consider the following factors:

1. **Accuracy**: Stanza generally provides better accuracy in sentence boundary detection compared to NLTK, especially for languages other than English.

2. **Language Support**: Stanza offers multilingual support out-of-the-box, making it suitable for applications that involve text in various languages. NLTK, on the other hand, may require additional configuration for certain languages.

3. **Performance**: NLTK is lightweight and easy to use, making it suitable for simple applications where performance is not a critical factor. However, for more demanding real-time applications or applications involving multiple languages, Stanza's performance and versatility make it a preferred choice.

In summary, choose the NLTK tokenizer for straightforward text-to-speech tasks in English or when simplicity is preferred. Opt for the Stanza tokenizer when working with multilingual text or when higher accuracy and performance are required.

## How to use voices

Every engine has a corresponding Voice class. So to AzureEngine there is AzureVoice, for ElevenlabsEngine ElevenlabsVoice exists etc. All these Voice classes have a "name" parameter implemented, which you can use to set the voice on the engine (or to show the name of the voice to the user).

### Retrieve voices

Every engine has a method named get_voices() implemented, which returns a list of objects of the voice class depending to the engine. So if you call get_voices() on an instance of the AzureEngine for example, you would get a list of AzureVoice objects representing the available voices for this engine.

### Set voices

Every engine also has a method named set_voice, which takes either a string with a voice name (or a part of a voice name) or an instance of the voice class of this engine.

## Script won't terminate

When you are using the CoquiEngine you need to call the shutdown method before closing the application to close multiprocessing pipe connections and terminate the worker process.
