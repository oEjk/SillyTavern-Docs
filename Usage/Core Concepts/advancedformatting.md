# Advanced Formatting

The settings provided in this section allow for more control over the prompt building strategy. Most specifics of the prompt building depend on whether a Pygmalion model is selected or special formatting is force-enabled. The core differences between the formatting schemas are listed below.

### Custom Chat Separator

Overrides the default separators controlled by "Disable example chats formatting" and "Disable chat start formatting" options (see below).

### For *Pygmalion* formatting

#### Disable description formatting

`**NAME's Persona:**`won't be prepended to the content of your character's Description box.

#### Disable scenario formatting

`**Scenario:**`won't be prepended to the content of your character's Scenario box.

#### Disable personality formatting

`**Personality:**`won't be prepended to the content of your character's Personality box.

#### Disable example chats formatting

`<START>` won't be added at the beginning of each example message block.
*(If custom separator is not set)*

#### Disable chat start formatting

`<START>` won't be added between the character card and the chat log.
*(If custom separator is not set)*

#### Always add character's name to prompt

Doesn't do anything (Included in Pygmalion formatting).

### For *non-Pygmalion* formatting

#### Disable description formatting

Has no effect.

#### Disable scenario formatting

`**Circumstances and context of the dialogue:**`won't be prepended to the content of your character's Scenario box.

#### Disable personality formatting

`**NAME's personality:**`won't be prepended to the content of your character's Personality box.

#### Disable example chats formatting

`This is how **Character** should talk` won't be added at the beginning of each example message block.
*(If custom separator is not set)*

#### Disable chat start formatting

`Then the roleplay chat between **User** and **Character** begins` won't be added between the character card and the chat log.
*(If custom separator is not set)*

#### Always add character's name to prompt

Appends character's name to the prompt to force the model to complete the message as the character:

```
** OTHER CONTEXT HERE **
Character:
```

### Tokenizer

A tokenizer is a tool that breaks down a piece of text into smaller units called tokens. These tokens can be individual words or even parts of words, such as prefixes, suffixes, or punctuation. A rule of thumb is that one token generally corresponds to 3~4 characters of text.

SillyTavern provides a "Best match" option that tries to match the tokenizer using the following rules depending on the API provider used.

Text Completion APIs **(overridable)**:
1. NovelAI Krake/Euterpe: GPT-2/3 tokenizer.
2. NovelAI Clio: NerdStash tokenizer.
3. NovelAI Kayra: NerdStash v2 tokenizer.
4. TextGen / KoboldAI / AI Horde: LLaMA tokenizer.

If you get inaccurate results or wish to experiment, you can set an *override tokenizer* for SillyTavern to use while forming a request to the AI backend:

1. None. Each token is estimated to be ~3.3 characters, rounded up to the nearest integer. **Try this if your prompts get cut off on high context lengths.** This approach is used by KoboldAI Lite.
2. GPT-3 tokenizer. **Used by pre-Turbo OpenAI models (ada, babbage, curie, davinci).** Can be previewed here: [OpenAI Tokenizer](https://platform.openai.com/tokenizer).
3. (Legacy) GPT-2/3 tokenizer. Used by original TavernAI. **Pick this if you're unsure.** More info: [gpt-2-3-tokenizer](https://github.com/josephrocca/gpt-2-3-tokenizer).
4. LLaMA tokenizer. Used by LLaMA 1/2 models family: Vicuna, Hermes, Airoboros, etc. **Pick if you use a LLaMA 1/2 model.**
5. NerdStash tokenizer. Used by NovelAI's Clio model. **Pick if you use the Clio model.**
6. NerdStash v2 tokenizer. Used by NovelAI's Kayra model. **Pick if you use the Kayra model.**
7. API tokenizer. Queries the generation API to get the token count directly from the model. Only supported by Oobabooga's TextGen. **Pick if you use the latest version of TextGen API.**

Chat Completion APIs **(non-overridable)**:
1. OpenAI / Claude / OpenRouter / Window: model-dependant tokenizer via [tiktoken](https://github.com/openai/tiktoken).
2. Scale API: GPT-4 tokenizer.
3. Fallback tokenizer (for proxies): GPT-3.5 turbo tokenizer.

### Token Padding

**Important: This section doesn't apply to OpenAI API. SillyTavern will always use a matching tokenizer for OpenAI models.**

SillyTavern cannot use a proper tokenizer provided by the model running on a remote instance of KoboldAI or Oobabooga's TextGen, so all token counts assumed during prompt generation are estimated based on the selected [tokenizer](#tokenizer) type.

Since the results of tokenization can be inaccurate on context sizes close to the model-defined maximum, some parts of the prompt may be trimmed or dropped, which may negatively affect the coherence of character definitions.

To prevent this, SillyTavern allocates a portion of the context size as padding to avoid adding more chat items than the model can accommodate. If you find that some part of the prompt is trimmed even with the most-matching tokenizer selected, adjust the padding so the description is not truncated.

You can input negative values for reverse padding, which allows allocating more than the set maximum amount of tokens.

### Multigen

*This feature provides a pseudo-streaming functionality which conflicts with token streaming. When Multigen is enabled and generation API supports streaming, only Multigen streaming will be used.*

SillyTavern tries to create faster and longer responses by chaining the generation using smaller batches.

#### Default settings

First batch = 50 tokens

Next batches = 30 tokens

#### Algorithm

1. Generate the first batch (if amount of generation setting is more than batch length).
2. Generate next batch of tokens until one of the stopping conditions is reached.
3. Append the generated text to the next cycle's prompt.

#### Stopping conditions

1. Generated enough text.
2. Character starts speaking for You.
3. &lt;|endoftext|&gt; token reached.
4. No text generated.
5. Stop sequence generated. (Instruct mode only)
