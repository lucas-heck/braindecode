.. _data_summary:
.. raw:: html

   <style>
    .tag {
      display: inline-block;
      background-color: #e0f7fa;
      color: #00796b;
      padding: 4px 8px;
      margin: 2px;
      border-radius: 4px;
      font-size: 0.9em;
    }
    .dataTables_filter {
      display: none;
    }
   </style>
   <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
   <link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/v/bm/dt-1.13.4/datatables.min.css"/>


Models Summary
~~~~~~~~~~~~~~~

This page offers a summary of many implemented models. Please note that this list may not be exhaustive. For the definitive and most current list, including detailed class documentation, please consult the :doc:`API documentation <api>`.

We are continually expanding this collection and welcome contributions! If you have implemented a model relevant to EEG, EcoG, or MEG analysis, consider adding it to Braindecode. See the "Submit a New Model" section below for details.

.. figure:: _static/model/models_analysis.png
   :alt: Braindecode Models
   :align: center

   Visualization comparing the models based on their total number of parameters (left plot) and the primary experimental they were designed for (right plot).

Columns definitions:
    - **Model**: The name of the model.
    - **Paradigm**: The paradigm(s) the model is typically used for (e.g., Motor Imagery, P300, Sleep Staging). 'General' indicates applicability across multiple paradigms or no specific paradigm focus.
    - **Type**: The model's primary function (e.g., Classification, Regression, Embedding).
    - **Freq (Hz)**: The data sampling rate (in Hertz) the model is designed for. Note that this might be adaptable depending on the specific dataset and application.
    - **Hyperparameters**: The mandatory hyperparameters required for instantiating the model class. These may include `n_chans` (number of channels), `n_outputs` (number of output classes or regression targets), `n_times` (number of time points in the input window), or `sfreq` (sampling frequency). Also, `n_times` can be derived implicitly by providing both `sfreq` and `input_window_seconds`.
    - **#Parameters**: The approximate total number of trainable parameters in the model, calculated using a consistent configuration (see note below).

.. raw:: html
   :file: generated/models_summary_table.html

The parameter counts shown in the table were calculated using consistent hyperparameters for models within the same paradigm, based largely on Braindecode's default implementation values. These counts provide a relative comparison but may differ from those reported in the original publications due to variations in specific architectural details, input dimensions used in the paper, or calculation methods.


Submit a new model
~~~~~~~~~~~~~~~~~~~~

Want to contribute a new model to Braindecode? Great! You can propose a new model by opening an `issue <https://github.com/braindecode/braindecode/issues>`__ (please include a link to the relevant publication or description) or, even better, directly submit your implementation via a `pull request <https://github.com/braindecode/braindecode/pulls>`__. We appreciate your contributions to expanding the library!

.. raw:: html

   <script type="text/javascript" src="https://cdn.datatables.net/v/bm/dt-1.13.4/datatables.min.js"></script>
   <script type="text/javascript">
    $(document).ready(function() {
      var table = $('.sortable').DataTable({
        "paging": false,
        "searching": true,
        "info": false,
      });

      var indexes = { Paradigm: -1, Type: -1, '#Parameters': -1 };
      table.columns().every(function(index) {
        var header = $(this.header()).text().trim();
        if (header in indexes) indexes[header] = index;
      });

      var uniqueParadigms = [...new Set(
        table.column(indexes.Paradigm).nodes().toArray().flatMap(cell =>
          [...$(cell).find('span.tag')].map(span => $(span).text())
        )
      )].sort();
      var uniqueTypes = [...new Set(
        table.column(indexes.Type).data().join(', ').split(', ').map(s => s.trim())
      )].sort();
      var paramsOptions = ['<1K', '<10K', '<100K', '<1M', '<10M', '<100M'];
      var paramsRanges = { '<1K': 1e3, '<10K': 1e4, '<100K': 1e5, '<1M': 1e6, '<10M': 1e7, '<100M': 1e8 };

      function createDropdown(options) {
        return $('<select>').append('<option value="All">All</option>').append(
          options.map(opt => `<option value="${opt}">${opt}</option>`)
        );
      }

      for (var col in indexes) {
        var header = table.column(indexes[col]).header();
        var text = $(header).text();
        $(header).html(`<span>${text}</span><br>`).append(
          createDropdown(col === '#Parameters' ? paramsOptions : (col === 'Paradigm' ? uniqueParadigms : uniqueTypes))
        );
      }

      var filters = { Paradigm: null, Type: null, '#Parameters': null };
      $.fn.dataTable.ext.search.push(function(settings, data, dataIndex) {
        var paradigms = [...$(table.row(dataIndex).node()).find('td').eq(indexes.Paradigm).find('span.tag')].map(span => $(span).text());
        var types = data[indexes.Type].split(', ').map(s => s.trim());
        var n_params = parseParams(data[indexes['#Parameters']]);

        return (!filters.Paradigm || paradigms.includes(filters.Paradigm)) &&
               (!filters.Type || types.some(type => filters.Type === type)) &&
               (!filters['#Parameters'] || (n_params !== null && n_params < filters['#Parameters']));
      });

      $('th select').on('change', function(e) {
        var col = $(this).closest('th').find('span').text();
        var val = $(this).val();
        filters[col] = val === 'All' ? null : (col === '#Parameters' ? paramsRanges[val] : val);
        table.draw();
        e.stopPropagation();
      }).on('click', e => e.stopPropagation());

      function parseParams(text) {
        text = text.trim().replace(/,/g, '');
        if (!text) return null;
        if (text.endsWith('K')) return parseFloat(text.slice(0, -1)) * 1e3;
        if (text.endsWith('M')) return parseFloat(text.slice(0, -1)) * 1e6;
        return parseFloat(text);
      }
    });
   </script>
