Skip to content 
Version Notice
This documentation is ahead of the last release by 8 commits. You may see documentation for features not yet supported in the latest release v0.0.24 2025-02-12. 
# `pydantic_ai.models.mistral`
## Setup
For details on how to set up authentication with this model, see model configuration for Mistral.
###  LatestMistralModelNames `module-attribute`
```
LatestMistralModelNames = Literal[
  "mistral-large-latest",
  "mistral-small-latest",
  "codestral-latest",
  "mistral-moderation-latest",
]

```

Latest Mistral models.
###  MistralModelName `module-attribute`
```
MistralModelName = Union[str, LatestMistralModelNames]

```

Possible Mistral model names.
Since Mistral supports a variety of date-stamped models, we explicitly list the most popular models but allow any name in the type hints. Since the Mistral docs for a full list.
###  MistralModelSettings
Bases: `ModelSettings`
Settings used for a Mistral model request.
Source code in `pydantic_ai_slim/pydantic_ai/models/mistral.py`
```
87
88
```
| ```
class MistralModelSettings(ModelSettings):
"""Settings used for a Mistral model request."""

```
  
---|---  
###  MistralModel `dataclass`
Bases: `Model`
A model that uses Mistral.
Internally, this uses the Mistral Python client to interact with the API.
API Documentation
Source code in `pydantic_ai_slim/pydantic_ai/models/mistral.py`
```
 93
 94
 95
 96
 97
 98
 99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
154
155
156
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
175
176
177
178
179
180
181
182
183
184
185
186
187
188
189
190
191
192
193
194
195
196
197
198
199
200
201
202
203
204
205
206
207
208
209
210
211
212
213
214
215
216
217
218
219
220
221
222
223
224
225
226
227
228
229
230
231
232
233
234
235
236
237
238
239
240
241
242
243
244
245
246
247
248
249
250
251
252
253
254
255
256
257
258
259
260
261
262
263
264
265
266
267
268
269
270
271
272
273
274
275
276
277
278
279
280
281
282
283
284
285
286
287
288
289
290
291
292
293
294
295
296
297
298
299
300
301
302
303
304
305
306
307
308
309
310
311
312
313
314
315
316
317
318
319
320
321
322
323
324
325
326
327
328
329
330
331
332
333
334
335
336
337
338
339
340
341
342
343
344
345
346
347
348
349
350
351
352
353
354
355
356
357
358
359
360
361
362
363
364
365
366
367
368
369
370
371
372
373
374
375
376
377
378
379
380
381
382
383
384
385
386
387
388
389
390
391
392
393
394
395
396
397
398
399
400
401
402
403
404
405
406
407
408
409
410
411
412
413
414
415
416
417
418
419
420
421
422
423
424
425
426
427
428
429
430
431
432
433
434
435
436
437
438
439
440
441
442
443
444
445
446
447
448
449
450
451
452
453
454
455
456
457
458
459
460
461
```
| ```
@dataclass(init=False)
class MistralModel(Model):
"""A model that uses Mistral.
  Internally, this uses the [Mistral Python client](https://github.com/mistralai/client-python) to interact with the API.
  [API Documentation](https://docs.mistral.ai/)
  """
  client: Mistral = field(repr=False)
  json_mode_schema_prompt: str = """Answer in JSON Object, respect the format:\n```\n{schema}\n```\n"""
  _model_name: MistralModelName = field(repr=False)
  _system: str | None = field(default='mistral', repr=False)
  def __init__(
    self,
    model_name: MistralModelName,
    *,
    api_key: str | Callable[[], str | None] | None = None,
    client: Mistral | None = None,
    http_client: AsyncHTTPClient | None = None,
    json_mode_schema_prompt: str = """Answer in JSON Object, respect the format:\n```\n{schema}\n```\n""",
  ):
"""Initialize a Mistral model.
    Args:
      model_name: The name of the model to use.
      api_key: The API key to use for authentication, if unset uses `MISTRAL_API_KEY` environment variable.
      client: An existing `Mistral` client to use, if provided, `api_key` and `http_client` must be `None`.
      http_client: An existing `httpx.AsyncClient` to use for making HTTP requests.
      json_mode_schema_prompt: The prompt to show when the model expects a JSON object as input.
    """
    self._model_name = model_name
    self.json_mode_schema_prompt = json_mode_schema_prompt
    if client is not None:
      assert http_client is None, 'Cannot provide both `mistral_client` and `http_client`'
      assert api_key is None, 'Cannot provide both `mistral_client` and `api_key`'
      self.client = client
    else:
      api_key = os.getenv('MISTRAL_API_KEY') if api_key is None else api_key
      self.client = Mistral(api_key=api_key, async_client=http_client or cached_async_http_client())
  async def request(
    self,
    messages: list[ModelMessage],
    model_settings: ModelSettings | None,
    model_request_parameters: ModelRequestParameters,
  ) -> tuple[ModelResponse, Usage]:
"""Make a non-streaming request to the model from Pydantic AI call."""
    check_allow_model_requests()
    response = await self._completions_create(
      messages, cast(MistralModelSettings, model_settings or {}), model_request_parameters
    )
    return self._process_response(response), _map_usage(response)
  @asynccontextmanager
  async def request_stream(
    self,
    messages: list[ModelMessage],
    model_settings: ModelSettings | None,
    model_request_parameters: ModelRequestParameters,
  ) -> AsyncIterator[StreamedResponse]:
"""Make a streaming request to the model from Pydantic AI call."""
    check_allow_model_requests()
    response = await self._stream_completions_create(
      messages, cast(MistralModelSettings, model_settings or {}), model_request_parameters
    )
    async with response:
      yield await self._process_streamed_response(model_request_parameters.result_tools, response)
  @property
  def model_name(self) -> MistralModelName:
"""The model name."""
    return self._model_name
  @property
  def system(self) -> str | None:
"""The system / model provider."""
    return self._system
  async def _completions_create(
    self,
    messages: list[ModelMessage],
    model_settings: MistralModelSettings,
    model_request_parameters: ModelRequestParameters,
  ) -> MistralChatCompletionResponse:
"""Make a non-streaming request to the model."""
    response = await self.client.chat.complete_async(
      model=str(self._model_name),
      messages=list(chain(*(self._map_message(m) for m in messages))),
      n=1,
      tools=self._map_function_and_result_tools_definition(model_request_parameters) or UNSET,
      tool_choice=self._get_tool_choice(model_request_parameters),
      stream=False,
      max_tokens=model_settings.get('max_tokens', UNSET),
      temperature=model_settings.get('temperature', UNSET),
      top_p=model_settings.get('top_p', 1),
      timeout_ms=self._get_timeout_ms(model_settings.get('timeout')),
      random_seed=model_settings.get('seed', UNSET),
    )
    assert response, 'A unexpected empty response from Mistral.'
    return response
  async def _stream_completions_create(
    self,
    messages: list[ModelMessage],
    model_settings: MistralModelSettings,
    model_request_parameters: ModelRequestParameters,
  ) -> MistralEventStreamAsync[MistralCompletionEvent]:
"""Create a streaming completion request to the Mistral model."""
    response: MistralEventStreamAsync[MistralCompletionEvent] | None
    mistral_messages = list(chain(*(self._map_message(m) for m in messages)))
    if (
      model_request_parameters.result_tools
      and model_request_parameters.function_tools
      or model_request_parameters.function_tools
    ):
      # Function Calling
      response = await self.client.chat.stream_async(
        model=str(self._model_name),
        messages=mistral_messages,
        n=1,
        tools=self._map_function_and_result_tools_definition(model_request_parameters) or UNSET,
        tool_choice=self._get_tool_choice(model_request_parameters),
        temperature=model_settings.get('temperature', UNSET),
        top_p=model_settings.get('top_p', 1),
        max_tokens=model_settings.get('max_tokens', UNSET),
        timeout_ms=self._get_timeout_ms(model_settings.get('timeout')),
        presence_penalty=model_settings.get('presence_penalty'),
        frequency_penalty=model_settings.get('frequency_penalty'),
      )
    elif model_request_parameters.result_tools:
      # Json Mode
      parameters_json_schemas = [tool.parameters_json_schema for tool in model_request_parameters.result_tools]
      user_output_format_message = self._generate_user_output_format(parameters_json_schemas)
      mistral_messages.append(user_output_format_message)
      response = await self.client.chat.stream_async(
        model=str(self._model_name),
        messages=mistral_messages,
        response_format={'type': 'json_object'},
        stream=True,
      )
    else:
      # Stream Mode
      response = await self.client.chat.stream_async(
        model=str(self._model_name),
        messages=mistral_messages,
        stream=True,
      )
    assert response, 'A unexpected empty response from Mistral.'
    return response
  def _get_tool_choice(self, model_request_parameters: ModelRequestParameters) -> MistralToolChoiceEnum | None:
"""Get tool choice for the model.
    - "auto": Default mode. Model decides if it uses the tool or not.
    - "any": Select any tool.
    - "none": Prevents tool use.
    - "required": Forces tool use.
    """
    if not model_request_parameters.function_tools and not model_request_parameters.result_tools:
      return None
    elif not model_request_parameters.allow_text_result:
      return 'required'
    else:
      return 'auto'
  def _map_function_and_result_tools_definition(
    self, model_request_parameters: ModelRequestParameters
  ) -> list[MistralTool] | None:
"""Map function and result tools to MistralTool format.
    Returns None if both function_tools and result_tools are empty.
    """
    all_tools: list[ToolDefinition] = (
      model_request_parameters.function_tools + model_request_parameters.result_tools
    )
    tools = [
      MistralTool(
        function=MistralFunction(name=r.name, parameters=r.parameters_json_schema, description=r.description)
      )
      for r in all_tools
    ]
    return tools if tools else None
  def _process_response(self, response: MistralChatCompletionResponse) -> ModelResponse:
"""Process a non-streamed response, and prepare a message to return."""
    assert response.choices, 'Unexpected empty response choice.'
    if response.created:
      timestamp = datetime.fromtimestamp(response.created, tz=timezone.utc)
    else:
      timestamp = _now_utc()
    choice = response.choices[0]
    content = choice.message.content
    tool_calls = choice.message.tool_calls
    parts: list[ModelResponsePart] = []
    if text := _map_content(content):
      parts.append(TextPart(content=text))
    if isinstance(tool_calls, list):
      for tool_call in tool_calls:
        tool = self._map_mistral_to_pydantic_tool_call(tool_call=tool_call)
        parts.append(tool)
    return ModelResponse(parts, model_name=response.model, timestamp=timestamp)
  async def _process_streamed_response(
    self,
    result_tools: list[ToolDefinition],
    response: MistralEventStreamAsync[MistralCompletionEvent],
  ) -> StreamedResponse:
"""Process a streamed response, and prepare a streaming response to return."""
    peekable_response = _utils.PeekableAsyncStream(response)
    first_chunk = await peekable_response.peek()
    if isinstance(first_chunk, _utils.Unset):
      raise UnexpectedModelBehavior('Streamed response ended without content or tool calls')
    if first_chunk.data.created:
      timestamp = datetime.fromtimestamp(first_chunk.data.created, tz=timezone.utc)
    else:
      timestamp = datetime.now(tz=timezone.utc)
    return MistralStreamedResponse(
      _response=peekable_response,
      _model_name=self._model_name,
      _timestamp=timestamp,
      _result_tools={c.name: c for c in result_tools},
    )
  @staticmethod
  def _map_mistral_to_pydantic_tool_call(tool_call: MistralToolCall) -> ToolCallPart:
"""Maps a MistralToolCall to a ToolCall."""
    tool_call_id = tool_call.id or None
    func_call = tool_call.function
    return ToolCallPart(func_call.name, func_call.arguments, tool_call_id)
  @staticmethod
  def _map_pydantic_to_mistral_tool_call(t: ToolCallPart) -> MistralToolCall:
"""Maps a pydantic-ai ToolCall to a MistralToolCall."""
    return MistralToolCall(
      id=t.tool_call_id,
      type='function',
      function=MistralFunctionCall(name=t.tool_name, arguments=t.args),
    )
  def _generate_user_output_format(self, schemas: list[dict[str, Any]]) -> MistralUserMessage:
"""Get a message with an example of the expected output format."""
    examples: list[dict[str, Any]] = []
    for schema in schemas:
      typed_dict_definition: dict[str, Any] = {}
      for key, value in schema.get('properties', {}).items():
        typed_dict_definition[key] = self._get_python_type(value)
      examples.append(typed_dict_definition)
    example_schema = examples[0] if len(examples) == 1 else examples
    return MistralUserMessage(content=self.json_mode_schema_prompt.format(schema=example_schema))
  @classmethod
  def _get_python_type(cls, value: dict[str, Any]) -> str:
"""Return a string representation of the Python type for a single JSON schema property.
    This function handles recursion for nested arrays/objects and `anyOf`.
    """
    # 1) Handle anyOf first, because it's a different schema structure
    if any_of := value.get('anyOf'):
      # Simplistic approach: pick the first option in anyOf
      # (In reality, you'd possibly want to merge or union types)
      return f'Optional[{cls._get_python_type(any_of[0])}]'
    # 2) If we have a top-level "type" field
    value_type = value.get('type')
    if not value_type:
      # No explicit type; fallback
      return 'Any'
    # 3) Direct simple type mapping (string, integer, float, bool, None)
    if value_type in SIMPLE_JSON_TYPE_MAPPING and value_type != 'array' and value_type != 'object':
      return SIMPLE_JSON_TYPE_MAPPING[value_type]
    # 4) Array: Recursively get the item type
    if value_type == 'array':
      items = value.get('items', {})
      return f'list[{cls._get_python_type(items)}]'
    # 5) Object: Check for additionalProperties
    if value_type == 'object':
      additional_properties = value.get('additionalProperties', {})
      additional_properties_type = additional_properties.get('type')
      if (
        additional_properties_type in SIMPLE_JSON_TYPE_MAPPING
        and additional_properties_type != 'array'
        and additional_properties_type != 'object'
      ):
        # dict[str, bool/int/float/etc...]
        return f'dict[str, {SIMPLE_JSON_TYPE_MAPPING[additional_properties_type]}]'
      elif additional_properties_type == 'array':
        array_items = additional_properties.get('items', {})
        return f'dict[str, list[{cls._get_python_type(array_items)}]]'
      elif additional_properties_type == 'object':
        # nested dictionary of unknown shape
        return 'dict[str, dict[str, Any]]'
      else:
        # If no additionalProperties type or something else, default to a generic dict
        return 'dict[str, Any]'
    # 6) Fallback
    return 'Any'
  @staticmethod
  def _get_timeout_ms(timeout: Timeout | float | None) -> int | None:
"""Convert a timeout to milliseconds."""
    if timeout is None:
      return None
    if isinstance(timeout, float):
      return int(1000 * timeout)
    raise NotImplementedError('Timeout object is not yet supported for MistralModel.')
  @classmethod
  def _map_user_message(cls, message: ModelRequest) -> Iterable[MistralMessages]:
    for part in message.parts:
      if isinstance(part, SystemPromptPart):
        yield MistralSystemMessage(content=part.content)
      elif isinstance(part, UserPromptPart):
        yield MistralUserMessage(content=part.content)
      elif isinstance(part, ToolReturnPart):
        yield MistralToolMessage(
          tool_call_id=part.tool_call_id,
          content=part.model_response_str(),
        )
      elif isinstance(part, RetryPromptPart):
        if part.tool_name is None:
          yield MistralUserMessage(content=part.model_response())
        else:
          yield MistralToolMessage(
            tool_call_id=part.tool_call_id,
            content=part.model_response(),
          )
      else:
        assert_never(part)
  @classmethod
  def _map_message(cls, message: ModelMessage) -> Iterable[MistralMessages]:
"""Just maps a `pydantic_ai.Message` to a `MistralMessage`."""
    if isinstance(message, ModelRequest):
      yield from cls._map_user_message(message)
    elif isinstance(message, ModelResponse):
      content_chunks: list[MistralContentChunk] = []
      tool_calls: list[MistralToolCall] = []
      for part in message.parts:
        if isinstance(part, TextPart):
          content_chunks.append(MistralTextChunk(text=part.content))
        elif isinstance(part, ToolCallPart):
          tool_calls.append(cls._map_pydantic_to_mistral_tool_call(part))
        else:
          assert_never(part)
      yield MistralAssistantMessage(content=content_chunks, tool_calls=tool_calls)
    else:
      assert_never(message)

```
  
---|---  
####  __init__
```
__init__(
  model_name: MistralModelName,
  *,
  api_key: str | Callable[[], str | None] | None = None,
  client: Mistral | None = None,
  http_client: AsyncClient | None = None,
  json_mode_schema_prompt: str = "Answer in JSON Object, respect the format:\n```\n{schema}\n```\n"
)

```

Initialize a Mistral model.
Parameters:
Name | Type | Description | Default  
---|---|---|---  
`model_name` |  `MistralModelName` |  The name of the model to use. |  _required_  
`api_key` |  `str | Callable[[], str | None] | None` |  The API key to use for authentication, if unset uses `MISTRAL_API_KEY` environment variable. |  `None`  
`client` |  `Mistral | None` |  An existing `Mistral` client to use, if provided, `api_key` and `http_client` must be `None`. |  `None`  
`http_client` |  `AsyncClient | None` |  An existing `httpx.AsyncClient` to use for making HTTP requests. |  `None`  
`json_mode_schema_prompt` |  `str` |  The prompt to show when the model expects a JSON object as input. |  `'Answer in JSON Object, respect the format:\n```\n{schema}\n```\n'`  
Source code in `pydantic_ai_slim/pydantic_ai/models/mistral.py`
```
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
```
| ```
def __init__(
  self,
  model_name: MistralModelName,
  *,
  api_key: str | Callable[[], str | None] | None = None,
  client: Mistral | None = None,
  http_client: AsyncHTTPClient | None = None,
  json_mode_schema_prompt: str = """Answer in JSON Object, respect the format:\n```\n{schema}\n```\n""",
):
"""Initialize a Mistral model.
  Args:
    model_name: The name of the model to use.
    api_key: The API key to use for authentication, if unset uses `MISTRAL_API_KEY` environment variable.
    client: An existing `Mistral` client to use, if provided, `api_key` and `http_client` must be `None`.
    http_client: An existing `httpx.AsyncClient` to use for making HTTP requests.
    json_mode_schema_prompt: The prompt to show when the model expects a JSON object as input.
  """
  self._model_name = model_name
  self.json_mode_schema_prompt = json_mode_schema_prompt
  if client is not None:
    assert http_client is None, 'Cannot provide both `mistral_client` and `http_client`'
    assert api_key is None, 'Cannot provide both `mistral_client` and `api_key`'
    self.client = client
  else:
    api_key = os.getenv('MISTRAL_API_KEY') if api_key is None else api_key
    self.client = Mistral(api_key=api_key, async_client=http_client or cached_async_http_client())

```
  
---|---  
####  request `async`
```
request(
  messages: list[ModelMessage],
  model_settings: ModelSettings | None,
  model_request_parameters: ModelRequestParameters,
) -> tuple[ModelResponse, Usage]

```

Make a non-streaming request to the model from Pydantic AI call.
Source code in `pydantic_ai_slim/pydantic_ai/models/mistral.py`
```
137
138
139
140
141
142
143
144
145
146
147
148
```
| ```
async def request(
  self,
  messages: list[ModelMessage],
  model_settings: ModelSettings | None,
  model_request_parameters: ModelRequestParameters,
) -> tuple[ModelResponse, Usage]:
"""Make a non-streaming request to the model from Pydantic AI call."""
  check_allow_model_requests()
  response = await self._completions_create(
    messages, cast(MistralModelSettings, model_settings or {}), model_request_parameters
  )
  return self._process_response(response), _map_usage(response)

```
  
---|---  
####  request_stream `async`
```
request_stream(
  messages: list[ModelMessage],
  model_settings: ModelSettings | None,
  model_request_parameters: ModelRequestParameters,
) -> AsyncIterator[StreamedResponse]

```

Make a streaming request to the model from Pydantic AI call.
Source code in `pydantic_ai_slim/pydantic_ai/models/mistral.py`
```
150
151
152
153
154
155
156
157
158
159
160
161
162
163
```
| ```
@asynccontextmanager
async def request_stream(
  self,
  messages: list[ModelMessage],
  model_settings: ModelSettings | None,
  model_request_parameters: ModelRequestParameters,
) -> AsyncIterator[StreamedResponse]:
"""Make a streaming request to the model from Pydantic AI call."""
  check_allow_model_requests()
  response = await self._stream_completions_create(
    messages, cast(MistralModelSettings, model_settings or {}), model_request_parameters
  )
  async with response:
    yield await self._process_streamed_response(model_request_parameters.result_tools, response)

```
  
---|---  
####  model_name `property`
```
model_name: MistralModelName

```

The model name.
####  system `property`
```
system: str | None

```

The system / model provider.
###  MistralStreamedResponse `dataclass`
Bases: `StreamedResponse`
Implementation of `StreamedResponse` for Mistral models.
Source code in `pydantic_ai_slim/pydantic_ai/models/mistral.py`
```
467
468
469
470
471
472
473
474
475
476
477
478
479
480
481
482
483
484
485
486
487
488
489
490
491
492
493
494
495
496
497
498
499
500
501
502
503
504
505
506
507
508
509
510
511
512
513
514
515
516
517
518
519
520
521
522
523
524
525
526
527
528
529
530
531
532
533
534
535
536
537
538
539
540
541
542
543
544
545
546
547
548
549
550
551
552
553
554
555
556
557
558
559
560
561
562
563
564
565
566
567
```
| ```
@dataclass
class MistralStreamedResponse(StreamedResponse):
"""Implementation of `StreamedResponse` for Mistral models."""
  _model_name: MistralModelName
  _response: AsyncIterable[MistralCompletionEvent]
  _timestamp: datetime
  _result_tools: dict[str, ToolDefinition]
  _delta_content: str = field(default='', init=False)
  async def _get_event_iterator(self) -> AsyncIterator[ModelResponseStreamEvent]:
    chunk: MistralCompletionEvent
    async for chunk in self._response:
      self._usage += _map_usage(chunk.data)
      try:
        choice = chunk.data.choices[0]
      except IndexError:
        continue
      # Handle the text part of the response
      content = choice.delta.content
      text = _map_content(content)
      if text:
        # Attempt to produce a result tool call from the received text
        if self._result_tools:
          self._delta_content += text
          maybe_tool_call_part = self._try_get_result_tool_from_text(self._delta_content, self._result_tools)
          if maybe_tool_call_part:
            yield self._parts_manager.handle_tool_call_part(
              vendor_part_id='result',
              tool_name=maybe_tool_call_part.tool_name,
              args=maybe_tool_call_part.args_as_dict(),
              tool_call_id=maybe_tool_call_part.tool_call_id,
            )
        else:
          yield self._parts_manager.handle_text_delta(vendor_part_id='content', content=text)
      # Handle the explicit tool calls
      for index, dtc in enumerate(choice.delta.tool_calls or []):
        # It seems that mistral just sends full tool calls, so we just use them directly, rather than building
        yield self._parts_manager.handle_tool_call_part(
          vendor_part_id=index, tool_name=dtc.function.name, args=dtc.function.arguments, tool_call_id=dtc.id
        )
  @property
  def model_name(self) -> MistralModelName:
"""Get the model name of the response."""
    return self._model_name
  @property
  def timestamp(self) -> datetime:
"""Get the timestamp of the response."""
    return self._timestamp
  @staticmethod
  def _try_get_result_tool_from_text(text: str, result_tools: dict[str, ToolDefinition]) -> ToolCallPart | None:
    output_json: dict[str, Any] | None = pydantic_core.from_json(text, allow_partial='trailing-strings')
    if output_json:
      for result_tool in result_tools.values():
        # NOTE: Additional verification to prevent JSON validation to crash in `_result.py`
        # Ensures required parameters in the JSON schema are respected, especially for stream-based return types.
        # Example with BaseModel and required fields.
        if not MistralStreamedResponse._validate_required_json_schema(
          output_json, result_tool.parameters_json_schema
        ):
          continue
        # The following part_id will be thrown away
        return ToolCallPart(tool_name=result_tool.name, args=output_json)
  @staticmethod
  def _validate_required_json_schema(json_dict: dict[str, Any], json_schema: dict[str, Any]) -> bool:
"""Validate that all required parameters in the JSON schema are present in the JSON dictionary."""
    required_params = json_schema.get('required', [])
    properties = json_schema.get('properties', {})
    for param in required_params:
      if param not in json_dict:
        return False
      param_schema = properties.get(param, {})
      param_type = param_schema.get('type')
      param_items_type = param_schema.get('items', {}).get('type')
      if param_type == 'array' and param_items_type:
        if not isinstance(json_dict[param], list):
          return False
        for item in json_dict[param]:
          if not isinstance(item, VALID_JSON_TYPE_MAPPING[param_items_type]):
            return False
      elif param_type and not isinstance(json_dict[param], VALID_JSON_TYPE_MAPPING[param_type]):
        return False
      if isinstance(json_dict[param], dict) and 'properties' in param_schema:
        nested_schema = param_schema
        if not MistralStreamedResponse._validate_required_json_schema(json_dict[param], nested_schema):
          return False
    return True

```
  
---|---  
####  model_name `property`
```
model_name: MistralModelName

```

Get the model name of the response.
####  timestamp `property`
```
timestamp: datetime

```

Get the timestamp of the response.
