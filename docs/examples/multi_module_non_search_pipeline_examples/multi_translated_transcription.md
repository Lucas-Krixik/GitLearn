<a href="https://colab.research.google.com/github/krixik-ai/krixik-docs/blob/main/docs/examples/multi_module_non_search_pipeline_examples/multi_translated_transcription.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

## Multi-Module Pipeline: Translated Transcription

This document details a modular pipeline that takes in an audio file, [`transcribes`](../../modules/ai_modules/transcribe_module.md) it, and [`translates`](../../modules/ai_modules/translate_module.md) the transcription into a desired language.

The document is divided into the following sections:

- [Pipeline Setup](#pipeline-setup)
- [Processing an Input File](#processing-an-input-file)

### Pipeline Setup

To achieve what we've described above, let's set up a pipeline sequentially consisting of the following modules:

- A [`transcribe`](../../modules/ai_modules/transcribe_module.md) module.

- A [`translate`](../../modules/ai_modules/translate_module.md) module.

We do this by leveraging the [`.create_pipeline`](../../system/pipeline_creation/create_pipeline.md) method, as follows:


```python
# create a pipeline as detailed above
pipeline = krixik.create_pipeline(name="multi_translated_transcription", module_chain=["transcribe", "translate"])
```

### Processing an Input File

Lets take a quick look at a short test file before processing.


```python
# examine contents of input file
import IPython

IPython.display.Audio(data_dir + "input/Interesting Facts About Colombia.mp3")
```





<audio  controls="controls" >
    Your browser does not support the audio element.
</audio>




Since the input audio is in English,  we'll use the default [`opus-mt-en-es`](https://huggingface.co/Helsinki-NLP/opus-mt-en-es) model of the [`translate`](../../modules/ai_modules/translate_module.md) module to translate the content into Spanish.

We will use the default models for every other module in the pipeline as well, so the [`modules`](../../system/parameters_processing_files_through_pipelines/process_method.md#selecting-models-via-the-modules-argument) argument of the [`.process`](../../system/parameters_processing_files_through_pipelines/process_method.md) method doesn't need to be leveraged.


```python
# process the file through the pipeline, as described above
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


```python
# nicely print the output of this process
print(json.dumps(process_output, indent=2))
```

    {
      "status_code": 200,
      "pipeline": "multi_translated_transcription",
      "request_id": "9e1cd6f1-7e3d-4e3f-8493-789b83164de1",
      "file_id": "b3b7d81b-2e81-4f53-806a-87e637e01b59",
      "message": "SUCCESS - output fetched for file_id b3b7d81b-2e81-4f53-806a-87e637e01b59.Output saved to location(s) listed in process_output_files.",
      "warnings": [],
      "process_output": [
        {
          "snippet": "Ese es un episodio mirando el gran pas de Columbia. Miramos algunos hechos realmente bsicos. Es el nombre, un poco de su historia, el tipo de gente que vive all, el tamao de la tierra y todo ese jazz. Pero en este video, vamos a entrar en un poco ms de una mirada detallada. Yo, qu est pasando chicos? Bienvenidos de nuevo a los hechos F2D. El canal donde miro las culturas y lugares de la gente, mi nombre es Dave Wouple. Y hoy vamos a mirar ms a Columbia en nuestro video de la Parte 2 de Columbia. Lo que me recuerda chicos, esto es parte de nuestra lista de Columbia. Lo pondr en el cuadro de descripcin de abajo y hablar ms sobre eso al final del video. Pero si eres nuevo aqu, nete a m todos los lunes para aprender sobre nuevos pases de todo el mundo. Usted puede hacer eso pulsando que suscribirse y ese botn de notificacin de cinturn. Pero eso es lo que pasa."
        }
      ],
      "process_output_files": [
        "../../../data/output/b3b7d81b-2e81-4f53-806a-87e637e01b59.json"
      ]
    }


To confirm that everything went as it should have, let's load in the text file output from `process_output_files`:


```python
# load in process output from file
with open(process_output["process_output_files"][0], "r") as file:
    print(file.read())
```

    [{"snippet": "Ese es un episodio mirando el gran pas de Columbia. Miramos algunos hechos realmente bsicos. Es el nombre, un poco de su historia, el tipo de gente que vive all, el tamao de la tierra y todo ese jazz. Pero en este video, vamos a entrar en un poco ms de una mirada detallada. Yo, qu est pasando chicos? Bienvenidos de nuevo a los hechos F2D. El canal donde miro las culturas y lugares de la gente, mi nombre es Dave Wouple. Y hoy vamos a mirar ms a Columbia en nuestro video de la Parte 2 de Columbia. Lo que me recuerda chicos, esto es parte de nuestra lista de Columbia. Lo pondr en el cuadro de descripcin de abajo y hablar ms sobre eso al final del video. Pero si eres nuevo aqu, nete a m todos los lunes para aprender sobre nuevos pases de todo el mundo. Usted puede hacer eso pulsando que suscribirse y ese botn de notificacin de cinturn. Pero eso es lo que pasa."}]
