## Yin & Yang: Demonstrating Complementary Provenance from noWorkflow & YesWorkflow

To demonstrate how [noWorkflow](https://github.com/gems-uff/noworkflow) (NW) and [YesWorkflow](https://github.com/yesworkflow-org/yw-prototypes) (YW) can be combined to answer provenance queries, we use a simulation of a crystallography experiment to collect X-ray diffraction data from a set of samples at a synchrotron radiation beam line. The script reads previously measured data quality statistics for each sample from an input spreadsheet, rejects samples that do not meet a minimum quality criterion, and, for the accepted samples, generates corrected images. Although this script uses meaningful variable names, it uses limited modularity. In fact, it has a single large top level function (simulate_data_collection) that performs several tasks. To cope with the code complexity, this function was annotated by a specialist user using YW annotation syntax. The complete annotated script is available at [simulate_data_collection.py](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/simulate_data_collection.py).

The following are the current instructions for the prototype demonstration:

1. Navigate to the project directory at [simulate_data_collection](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection).
Note that it contains the simulation script, [simulate_data_collection.py](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/simulate_data_collection.py); two input files, [cassette_q55_spreadsheet.csv](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/cassette_q55_spreadsheet.csv) and [calibration.img](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/calibration.img); and two directories, nw and yw, refering respectively to noWorkflow and YesWorkflow. In the [yw](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/yw) directory, the [yw.properties](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/yw/yw.properties) file configure YesWorkflow for this script.

1. Open the simulation script, [simulate_data_collection.py](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/simulate_data_collection.py). This file has many YW annotations. Note that [load_screening_results in line 49](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/simulate_data_collection.py#L49) receives *sample_spreadsheet* as input, and outputs *sample_name* and *sample_quality*.
The input is qualified by a @uri tag that speficies that *sample_spreadsheet* corresponds to a file in the given directory. Both outputs are used as params to  [calculate_strategy in line 63](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/simulate_data_collection.py#63), in addition to other params: *sample_score_cutoff* and *data_redundancy*.

1. Navigate to [yw](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/yw) directory and run `yw graph` to produce the prospective provenance as a DOT (Graphviz) file, [combined.gv](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/combined.gv). Use `dot combined.gv -Tpng -o yw-dataflow.png` to create a figure from the dot file, as shown below.

  ![prospective provenance generated by YW](https://github.com/gems-uff/yin-yang-demo/blob/master/figs/yw-dataflow.png)

  This figure has all blocks declared by YW annotations. Note that two outputs from load_screening_results are inputs to calculate_strategy.

1. YW generated [extractfacts.P](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb/extractfacts.P) and [modelfacts.P](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb/modelfacts.P) on [yw/xsb](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb) directory. Open them. The facts *annotation* on [extractfacts.P](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb/extractfacts.P) and *port* on [modelfacts.P](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb/modelfacts.P) are the most relevant to YW\*NW integration.

  ```prolog
    % extractfacts.P
    % FACT: annotation(annotation_id, source_id, line_number, tag, keyword, value).
    annotation(79, 1, 121, 'in', '@in', 'raw_image_path').

    % modelfacts.P
    % FACT: port(port_id, port_type, port_name, qualified_port_name, port_annotation_id, data_id).
    port(37, 'in', 'raw_image_path', 'simulate_data_collection.transform_images<-raw_image_path', 79, 25).
  ```

1. Open `xsb` on [yw/xsb](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb) directory.
   Load [rules.P](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/yw/xsb/rules.P), [extractfacts.P](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb/extractfacts.P), and [modelfacts.P](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb/modelfacts.P):

  ```prolog
  [rules].
  [extractfacts].
  [modelfacts].
  ```

   Run the following YW model (prospective) queries and confirm that the answers make sense.

  - What are the names of steps that comprise the top-level workflow implemented by the script?

    ```prolog
    top_workflow(W),
    has_subprogram(W, P),
    program(P, ProgramName, _, _, _).
    ```

  - What data is output by the collect_data_set step?

    ```prolog
    program(P, _, 'simulate_data_collection.collect_data_set', _, _),
    has_out_port(P, OUT),
    port_data(OUT, DataName, _),
    port_description(OUT,Description).
    ```

1. Go back to [simulate_data_collection](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection) directory. Run the script using noWorkflow

   `now run -e Tracer -d 3 simulate_data_collection.py q55 --cutoff 12 --redundancy 0`

   The option -e Tracer indicates to collect dependencies between variables, and the option -d 3 limits the collection depth.

1. Note that there is a [run](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/run) directory now. Run `tree run` and look at the created files. Note that the file [run/data/DRT322/DRT322_11000eV_002.img](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/run/data/DRT322/DRT322_11000eV_002.img) corresponds to the @uri tag at [line 123](https://github.com/gems-uff/yin-yang-demo/blob/master/simulate_data_collection/simulate_data_collection.py#L123)

1.  Run `now show -f` to get a list of files collected by NW.

  ```
  ...
    Name: run/data/DRT322/DRT322_11000eV_002.img
      Mode: wt
      Buffering: default
      Content hash before: None
      Content hash after: da39a3ee5e6b4b0d3255bfef95601890afd80709
      Timestamp: 2016-04-11 21:09:55.082741
      Function: simulate_data_collection -> transform_image -> __init__ ->  ... -> open
  ...
  ```

  Note that [run/data/DRT322/DRT322_11000eV_002.img](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/run/data/DRT322/DRT322_11000eV_002.img) were opened on a __init__ function, called by transform_image, called by the simulation_data_collection script itself.

1. Run `now export -r -m dependency > nw/kb.pl` to export NW provenance to prolog.

1. Navigate to [nw](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/nw) directory and load *kb.pl* on swipl: `swipl -s kb.pl`. Run the following prospective queries:

  - What functions does the top-level function call?

      ```prolog
      object(_, -1, _, Name, 'FUNCTION_CALL').
      ```
      Top-level function here refers to the script itself

  - Are any functions defined in the script not called by the top-level function?

      ```prolog
      findall(FunctionName, (function_def(_, _, FunctionName, _, _, _, _),
                             sub_atom(FunctionName, _, _, _, '.')), InternalFunctions),
      findall(FunctionName, function_def(_, _, FunctionName, _, _, _, _), AllFunctions),
      subtract(AllFunctions, InternalFunctions, TopLevelFunctions),
      findall(CallName, object(_, -1, _, CallName, 'FUNCTION_CALL'), Calls),
      subtract(TopLevelFunctions, Calls, Result).
      ```

1. Run `yw recon` on [yw](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw) directory, open [reconfacts.P](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb/reconfacts.P).

  The *resource* facts indicate files found after the execution, and the *data_resource* facts connect the resource facts to prospective data:

  ```prolog
  % FACT: resource(resource_id, resource_uri).
  resource(1, 'cassette_q55_spreadsheet.csv').

  % FACT: data_resource(data_id, resource_id).
  data_resource(4, 1).
  ```

1. Open `xsb` again on [yw/xsb](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/yw/xsb) directory. Load the following files:

  ```prolog
  [rules].
  [extractfacts].
  [modelfacts].
  [reconfacts].
  ```

  Answer the following retrospective queries using YW:

  - What samples did the run of the script collect images from?

    ```prolog
    data(D, _, 'simulate_data_collection[raw_image]'),
    data_resource(D, R),
    written_resource_metadata(R, 'sample_id', Sample).
    ```

  - Where is the raw image from which corrected image [run/data/DRT322/DRT322_10000eV_001.img](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/run/data/DRT322/DRT322_10000eV_001.img) is derived?

    ```prolog
    resource(R1, 'run/data/DRT322/DRT322_10000eV_001.img'),
    resource(R2, RawImageFile),
    depends_on(R1, R2).
    ```

1. Open swipl again on [nw](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/nw) directory: `swipl -s kb.pl`

  Answer the following retrospective queries using NW:

  - What files were written during calls to the *transform_image* function?

    ```prolog
    indirect_access(_, 'transform_image', FileName).
    ```

  - What variables carry values returned by the function *spreadsheet_rows* to calls to the *calculate_strategy*  function?

    ```prolog
    variable(_, ActivationId, VariableId, 'calculate_strategy', _, 'now(n/a)', _),
    slice(_, VariableId, Deps),
    findall(X, (variable(_, ActivationId, CalculateStrategyId, 'spreadsheet_rows(sample_spreadsheet)', _, 'now(n/a)', _),
                dep(_, X, CalculateStrategyId), member(X, Deps)), CalculateStrategyReturns),
    maplist(var_name(_), CalculateStrategyReturns, Result).
    ```

1. Generate a NW dataflow. Graphviz is not able to produce a graph with all variables captured by noWorkflow on depth 3. So instead of just producing the dataflow, go back to [simulate_data_collection](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection) directory, and run noWorkflow again with the collection limited to depth 2; then export the dataflow to png:

  ```
  now run -e Tracer -d 2 simulate_data_collection.py q55 --cutoff 12 --redundancy 0
  now dataflow | dot -Tpng -o nw/nw-dataflow.png
  ```

  The dataflow contains lots of details about functions that were called, variables and files, as shown below.
  ![dataflow generated by noWorkflow of the example script](https://github.com/gems-uff/yin-yang-demo/blob/master/figs/nw-dataflow.png)

  To make sense of this graph, we need to select what is important for the user. To do so, we use YW\*NW.

1. Run YW\*NW integration script, producing a *nw_yw_facts.P* file.

1. Lets say you want to see the full lineage of [run/data/DRT240/DRT240_11000eV_002.img](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/run/data/DRT240/DRT240_11000eV_002.img). Generate it with YW\*NW and compare to YW prospective view.

  ![lineage generated by YW\*NW](https://github.com/gems-uff/yin-yang-demo/blob/master/figs/yn-lineage.png)

1. Navigate to [nw](https://github.com/gems-uff/yin-yang-demo/blob/after_run/simulate_data_collection/nw) directory and run `python dataflow.py "run/data/DRT240/DRT240_11000eV_002.img" -o nw-lineage.dot && dot nw-lineage.dot -Tpng -o nw-lineage.png`. This command will produce the file  figure bellow:

  ![lineage generated by NW](https://github.com/gems-uff/yin-yang-demo/blob/master/figs/nw-lineage.png)

We expect that users that haven't seen the script get a higher understanding of the lineage through YW\*NW, data element names are more informative and the graph is less overwhelming. Note that YW\*NW image was based on a depth 3 knowledge base, while NW one was based on a depth 2. With a depth 3 knowledge base, NW graph would be even more overwhelming.
