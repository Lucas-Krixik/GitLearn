<a href="https://colab.research.google.com/github/krixik-ai/krixik-docs/blob/main/docs/examples/multi_module_non_search_pipeline_examples/multi_sentiment_analysis_on_translated_transcription.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

## Multi-Module Pipeline: Sentiment Analysis on Translated Transcription

This document details a modular pipeline that takes in an audio file in a non-English language, [`transcribes`](../../modules/ai_modules/transcribe_module.md) it, [`translates`](../../modules/ai_modules/translate_module.md) the transcript into English, and then performs [`sentiment analysis`](../../modules/ai_modules/sentiment_module.md) on each sentence of the translated transcript.

The document is divided into the following sections:

- [Pipeline Setup](#pipeline-setup)
- [Processing an Input File](#processing-an-input-file)

### Pipeline Setup

To achieve what we've described above, let's set up a pipeline sequentially consisting of the following modules:

- A [`transcribe`](../../modules/ai_modules/transcribe_module.md) module.

- A [`translate`](../../modules/ai_modules/translate_module.md) module.

- A [`json-to-txt`](../../modules/support_function_modules/json-to-txt_module.md) module.

- A [`parser`](../../modules/support_function_modules/parser_module.md) module.

- A [`sentiment`](../../modules/ai_modules/sentiment_module.md) module.

We use the [`json-to-txt`](../../modules/support_function_modules/json-to-txt_module.md) and [`parser`](../../modules/support_function_modules/parser_module.md) combination, which combines the transcribed snippets into one document and then splices it again, to make sure that any pauses in speech don't make for partial snippets that can confuse the [`sentiment`](../../modules/ai_modules/sentiment_module.md) model.

Pipeline setup is accomplished through the [`.create_pipeline`](../../system/pipeline_creation/create_pipeline.md) method, as follows:


```python
# create a pipeline as detailed above
pipeline = krixik.create_pipeline(
    name="multi_sentiment_analysis_on_translated_transcription", module_chain=["transcribe", "translate", "json-to-txt", "parser", "sentiment"]
)
```

### Processing an Input File

Lets take a quick look at a test file before processing. Given that we're [`translating`](../../modules/ai_modules/translate_module.md) before performing [`sentiment`](../../modules/ai_modules/sentiment_module.md), we'll start with a Spanish-language audio file.


```python
# examine contents of input file
import IPython

IPython.display.Audio(data_dir + "input/deadlift.mp3")
```





<audio  controls="controls" >
    Your browser does not support the audio element.
</audio>




Since the input audio is in Spanish, we'll use the (non-default) [`opus-mt-es-en`](https://huggingface.co/Helsinki-NLP/opus-mt-es-en) model of the [`translate`](../../modules/ai_modules/translate_module.md) module to translate its transcript into English. We will also leverage a stronger model than the [default](../../modules/ai_modules/transcribe_module.md#available-models-in-the-transcribe-module) for our [`transcription`](../../modules/ai_modules/transcribe_module.md).

We will use the default models for every other module in the pipeline as well, so they don't have to be specified in the [`modules`](../../system/parameters_processing_files_through_pipelines/process_method.md#selecting-models-via-the-modules-argument) argument of the [`.process`](../../system/parameters_processing_files_through_pipelines/process_method.md) method.


```python
# process the file through the pipeline, as described above
process_output = pipeline.process(
    local_file_path=data_dir + "input/deadlift.mp3",  # the initial local filepath where the input file is stored
    local_save_directory=data_dir + "output",  # the local directory that the output file will be saved to
    expire_time=60 * 30,  # process data will be deleted from the Krixik system in 30 minutes
    wait_for_process=True,  # wait for process to complete before returning IDE control to user
    verbose=False,  # do not display process update printouts upon running code
    modules={"transcribe": {"model": "whisper-base"}, "translate": {"model": "opus-mt-es-en"}},
)  # specify a non-default model for use in two modules whose type is only present once each in the pipeline (otherwise, would have to refer to them positionally)
```

The output of this process is printed below. To learn more about each component of the output, review documentation for the [`.process`](../../system/parameters_processing_files_through_pipelines/process_method.md) method.

Because the output of this particular module-model pair is a JSON file, the process output is provided in this object as well (this is only the case for JSON outputs).  Moreover, the output file itself has been saved to the location noted in the `process_output_files` key.  The `file_id` of the processed input is used as a filename prefix for the output file.


```python
# nicely print the output of this process
print(json.dumps(process_output, indent=2))
```

    {
      "status_code": 200,
      "pipeline": "multi_sentiment_analysis_on_translated_transcription",
      "request_id": "fde9bc3e-5e78-4c72-9f9e-48f2387a7e61",
      "file_id": "6c7c8bf3-8288-40ef-880c-c829f7a39839",
      "message": "SUCCESS - output fetched for file_id 6c7c8bf3-8288-40ef-880c-c829f7a39839.Output saved to location(s) listed in process_output_files.",
      "warnings": [],
      "process_output": [
        {
          "snippet": "The first one is to start with the feet between the width of the hips and the Shoulders, the heels in the ground, a neutral column, medium-potential abdomen, the Shoulders, let's go, looking at them, scholarship the Shoulders are lightly in front of the bar or on the Shoulders, the arms, the right arms, the symmetric hands and an anchored enough to not rather the knees and we can have a lightly front look.",
          "positive": 0.903,
          "negative": 0.097,
          "neutral": 0.0
        },
        {
          "snippet": "To make movement, we will push the heels from the head, we will start to raise the hips and the Shoulders together,",
          "positive": 0.989,
          "negative": 0.011,
          "neutral": 0.0
        }
      ],
      "process_output_files": [
        "../../../data/output/6c7c8bf3-8288-40ef-880c-c829f7a39839.json"
      ]
    }


To confirm that everything went as it should have, let's load in the text file output from `process_output_files`:


```python
# load in process output from file
with open(process_output["process_output_files"][0]) as f:
    print(json.dumps(json.load(f), indent=2))
```

    [
      {
        "snippet": "The first one is to start with the feet between the width of the hips and the Shoulders, the heels in the ground, a neutral column, medium-potential abdomen, the Shoulders, let's go, looking at them, scholarship the Shoulders are lightly in front of the bar or on the Shoulders, the arms, the right arms, the symmetric hands and an anchored enough to not rather the knees and we can have a lightly front look.",
        "positive": 0.903,
        "negative": 0.097,
        "neutral": 0.0
      },
      {
        "snippet": "To make movement, we will push the heels from the head, we will start to raise the hips and the Shoulders together,",
        "positive": 0.989,
        "negative": 0.011,
        "neutral": 0.0
      }
    ]
