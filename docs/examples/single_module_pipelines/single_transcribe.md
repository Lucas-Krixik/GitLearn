<a href="https://colab.research.google.com/github/krixik-ai/krixik-docs/blob/main/docs/examples/single_module_pipelines/single_transcribe.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

## Single-Module Pipeline: `transcribe`

This document is a walkthrough of how to assemble and use a single-module pipeline that only includes a [`transcribe`](../../modules/ai_modules/transcribe_module.md) module. It is divided into the following sections:

- [Pipeline Setup](#pipeline-setup)
- [Required Input Format](#required-input-format)
- [Using the Default Model](#using-the-default-model)
- [Using a Non-Default Model](#using-a-non-default-model)

### Pipeline Setup

Let's first instantiate a single-module [`transcribe`](../../modules/ai_modules/transcribe_module.md)  pipeline.

We use the [`.create_pipeline`](../../system/pipeline_creation/create_pipeline.md) method for this, passing only the [`transcribe`](../../modules/ai_modules/transcribe_module.md)  module name into `module_chain`.


```python
# create a pipeline with a single transcribe module
pipeline = krixik.create_pipeline(name="single_transcribe_1", module_chain=["transcribe"])
```

### Required Input Format

The [`transcribe`](../../modules/ai_modules/transcribe_module.md)  module accepts audio inputs. Acceptable file formats are only MP3 for the time being.

Let's take a quick look at a valid input file, and then process it.


```python
# examine contents of input file
import IPython

IPython.display.Audio(data_dir + "input/Interesting Facts About Colombia.mp3")
```





<audio  controls="controls" >
    Your browser does not support the audio element.
</audio>




### Using the Default Model

Let's process our test input file using the [`transcribe`](../../modules/ai_modules/transcribe_module.md)  module's [default model](../../modules/ai_modules/transcribe_module.md#available-models-in-the-transcribe-module) : [`whisper-tiny`](https://huggingface.co/openai/whisper-tiny).

Given that this is the default model, we need not specify model selection through the optional [`modules`](../../system/parameters_processing_files_through_pipelines/process_method.md#selecting-models-via-the-modules-argument) argument in the [`.process`](../../system/parameters_processing_files_through_pipelines/process_method.md) method.


```python
# process the file with the default model
process_output = pipeline.process(
    local_file_path=data_dir + "input/Interesting Facts About Colombia.mp3",  # the initial local filepath where the input file is stored
    local_save_directory=data_dir + "output",  # the local directory that the output file will be saved to
    expire_time=60 * 30,  # process data will be deleted from the Krixik system in 30 minutes
    wait_for_process=True,  # wait for process to complete before returning IDE control to user
    verbose=False,
)  # do not display process update printouts upon running code
```

The output of this process is printed below. To learn more about each component of the output, review documentation for the [`.process`](../../system/parameters_processing_files_through_pipelines/process_method.md) method.

Because the output of this particular module-model pair is a JSON file, the process output is provided in this object as well (this is only the case for JSON outputs).  Moreover, the output file itself has been saved to the location noted in the `process_output_files` key.  The `file_id` of the processed input is used as a filename prefix for the output file.

To confirm that everything went as it should have, let's load in the text file output from `process_output_files`:


```python
# load in process output from file - here we only print the transcript, and not the timestamped version, since the output is quite long
with open(process_output["process_output_files"][0]) as f:
    print(json.dumps(json.load(f)[0]["transcript"].strip(), indent=2))
```

    "That's episode looking at the great country of Columbia. We looked at some really basic facts. It's name, a bit of its history, the type of people that live there, land size and all that jazz. But in this video, we're gonna go into a little bit more of a detailed look. Yo, what is going on guys? Welcome back to F2D facts. The channel where I look at people cultures and places, my name is Dave Wouple. And today we are gonna be looking more at Columbia in our Columbia Part 2 video. Which just reminds me guys, this is part of our Columbia playlist. I'll put it down in the description box below and I'll talk about that more at the end of the video. But if you're new here, join me every single Monday to learn about new countries from around the world. You can do that by hitting that subscribe and that belt notification button. But that skits."


As anticipated, the returned JSON file has not only the snippets of transcribed text, but along with each includes timestamps and a "confidence" value for the accuracy of each transcription.

### Using a Non-Default Model

To use a [non-default model](../../modules/ai_modules/transcribe_module.md#available-models-in-the-transcribe-module) like [`whisper-large-v3`](https://huggingface.co/openai/whisper-large-v3), we must enter it explicitly through the [`modules`](../../system/parameters_processing_files_through_pipelines/process_method.md#selecting-models-via-the-modules-argument) argument when invoking the [`.process`](../../system/parameters_processing_files_through_pipelines/process_method.md) method.

We do so below to process the same input file shown above.


```python
# process the file with a non-default model
process_output = pipeline.process(
    local_file_path=data_dir + "input/Interesting Facts About Colombia.mp3",  # all parameters save 'modules' as above
    local_save_directory=data_dir + "output",
    expire_time=60 * 30,
    wait_for_process=True,
    verbose=False,
    modules={"transcribe": {"model": "whisper-large-v3"}},
)  # specify a non-default model for this process as well as its parameters
```


```python
# load in process output from file - here we only print the transcript, and not the timestamped version, since the output is quite long
with open(process_output["process_output_files"][0]) as f:
    print(json.dumps(json.load(f)[0]["transcript"].strip(), indent=2))
```