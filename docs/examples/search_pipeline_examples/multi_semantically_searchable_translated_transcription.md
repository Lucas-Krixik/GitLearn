<a href="https://colab.research.google.com/github/krixik-ai/krixik-docs/blob/main/docs/examples/search_pipeline_examples/multi_semantically_searchable_translated_transcription.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

## Multi-Module Pipeline: Semantically-Searchable Translated Transcription

This document details a modular pipeline that takes in an audio file, [`transcribes`](../../modules/ai_modules/transcribe_module.md) it, [`translates`](../../modules/ai_modules/translate_module.md) the transcription into a desired language, and makes the result [`semantically searchable`](../../system/search_methods/semantic_search_method.md).

The document is divided into the following sections:

- [Pipeline Setup](#pipeline-setup)
- [Processing an Input File](#processing-an-input-file)
- [Performing Semantic Search](#performing-semantic-search)

### Pipeline Setup

To achieve what we've described above, let's set up a pipeline sequentially consisting of the following modules:

- A [`transcribe`](../../modules/ai_modules/transcribe_module.md) module.

- A [`translate`](../../modules/ai_modules/translate_module.md) module.

- A [`json-to-txt`](../../modules/support_function_modules/json-to-txt_module.md) module.

- A [`parser`](../../modules/support_function_modules/parser_module.md) module.

- A [`text-embedder`](../../modules/ai_modules/text-embedder_module.md) module.

- A [`vector-db`](../../modules/database_modules/vector-db_module.md) module.

We use the [`json-to-txt`](../../modules/support_function_modules/json-to-txt_module.md) and [`parser`](../../modules/support_function_modules/parser_module.md) combination, which combines the transcribed snippets into one document and then splices it again, to make sure that any pauses in speech don't make for partial snippets that can confuse the [`text-embedder`](../../modules/ai_modules/text-embedder_module.md) model.

Pipeline setup is accomplished through the [`.create_pipeline`](../../system/pipeline_creation/create_pipeline.md) method, as follows:


```python
# create a pipeline as detailed above
pipeline = krixik.create_pipeline(
    name="multi_semantically_searchable_translated_transcription",
    module_chain=["transcribe", "translate", "json-to-txt", "parser", "text-embedder", "vector-db"],
)
```

### Processing an Input File

Lets take a quick look at a test file before processing.


```python
# examine contents of input file
import IPython

IPython.display.Audio(data_dir + "input/deadlift.mp3")
```





<audio  controls="controls" >
    Your browser does not support the audio element.
</audio>




Since the input text is in Spanish, we'll use the (non-default) [`opus-mt-es-en`](https://huggingface.co/Helsinki-NLP/opus-mt-es-en) model of the [`translate`](../../modules/ai_modules/translate_module.md) module to translate it into English.

We will use the default models for every other module in the pipeline, so they don't have to be specified in the [`modules`](../../system/parameters_processing_files_through_pipelines/process_method.md#selecting-models-via-the-modules-argument) argument of the [`.process`](../../system/parameters_processing_files_through_pipelines/process_method.md) method.


```python
# process the file through the pipeline, as described above
process_output = pipeline.process(
    local_file_path=data_dir + "input/deadlift.mp3",  # the initial local filepath where the input file is stored
    local_save_directory=data_dir + "output",  # the local directory that the output file will be saved to
    expire_time=60 * 30,  # process data will be deleted from the Krixik system in 30 minutes
    wait_for_process=True,  # wait for process to complete before returning IDE control to user
    verbose=False,  # do not display process update printouts upon running code
    modules={"translate": {"model": "opus-mt-es-en"}},
)  # specify a non-default model for use in the translate module
```

The output of this process is printed below. To learn more about each component of the output, review documentation for the [`.process`](../../system/parameters_processing_files_through_pipelines/process_method.md) method.

Because the output of this particular module-model pair is a [FAISS](https://github.com/facebookresearch/faiss) database file, `process_output` is "null". However, the output file has been saved to the location noted in the `process_output_files` key.  The `file_id` of the processed input is used as a filename prefix for the output file.


```python
# nicely print the output of this process
print(json.dumps(process_output, indent=2))
```

    {
      "status_code": 200,
      "pipeline": "multi_semantically_searchable_translated_transcription",
      "request_id": "80906843-ff56-49a0-9e1f-e6175837d80b",
      "file_id": "afafa5e7-33e7-4584-8b1e-9477f0f4c8ab",
      "message": "SUCCESS - output fetched for file_id afafa5e7-33e7-4584-8b1e-9477f0f4c8ab.Output saved to location(s) listed in process_output_files.",
      "warnings": [],
      "process_output": null,
      "process_output_files": [
        "../../../data/output/afafa5e7-33e7-4584-8b1e-9477f0f4c8ab.faiss"
      ]
    }


### Performing Semantic Search

Krixik's [`.semantic_search`](../../system/search_methods/semantic_search_method.md) method enables semantic search on documents processed through certain pipelines. Given that the [`.semantic_search`](../../system/search_methods/semantic_search_method.md) method both [embeds](../../modules/ai_modules/text-embedder_module.md) the query and performs the search, it can only be used with pipelines containing both a [`text-embedder`](../../modules/ai_modules/text-embedder_module.md) module and a [`vector-db`](../../modules/database_modules/vector-db_module.md) module in immediate succession.

Since our pipeline satisfies this condition, it has access to the [`.semantic_search`](../../system/search_methods/semantic_search_method.md) method. Let's use it to query our text with natural language, as shown below:


```python
# perform semantic_search over the file in the pipeline
semantic_output = pipeline.semantic_search(query="be sure to hold the weights very firmly", file_ids=[process_output["file_id"]])

# nicely print the output of this process
print(json.dumps(semantic_output, indent=2))
```

    {
      "status_code": 200,
      "request_id": "b63645fd-d5f0-4f4e-ab96-8ff017365341",
      "message": "Successfully queried 1 user file.",
      "warnings": [],
      "items": [
        {
          "file_id": "afafa5e7-33e7-4584-8b1e-9477f0f4c8ab",
          "file_metadata": {
            "file_name": "krixik_generated_file_name_xdbqgopkac.mp3",
            "symbolic_directory_path": "/etc",
            "file_tags": [],
            "num_vectors": 4,
            "created_at": "2024-06-05 14:55:05",
            "last_updated": "2024-06-05 14:55:05"
          },
          "search_results": [
            {
              "snippet": "To make movement, we have to put the columns in the legs, we are going to start to look at the cauderas and the men together.",
              "line_numbers": [
                1
              ],
              "distance": 0.327
            },
            {
              "snippet": "To begin, we want to see the feet in the anchors of the caudera and the men, the columns in the ground, a neutral column, mediated by the abdomen, the men are going to go through there.",
              "line_numbers": [
                1
              ],
              "distance": 0.381
            },
            {
              "snippet": "The men are lightly in front of the bar or in the top, the men are symmetric and sufficient to not be in the ground.",
              "line_numbers": [
                1
              ],
              "distance": 0.381
            },
            {
              "snippet": "We can have a look at the front.",
              "line_numbers": [
                1
              ],
              "distance": 0.445
            }
          ]
        }
      ]
    }
