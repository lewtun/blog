---
keywords: fastai
title: Recovering columns hidden by the 🤗  Trainer
comments: false
categories: [til,nlp,huggingface,transformers]
badges: true
nb_path: _notebooks/2021-01-15-til-recovering-hidden-trainer-columns.ipynb
layout: notebook
---

<!--
#################################################
### THIS FILE WAS AUTOGENERATED! DO NOT EDIT! ###
#################################################
# file to edit: _notebooks/2021-01-15-til-recovering-hidden-trainer-columns.ipynb
-->

<div class="container" id="notebook-container">

    {% raw %}

<div class="cell border-box-sizing code_cell rendered">

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Lately, I've been using the <code>transformers</code> trainer together with the <code>datasets</code> library and I was a bit mystified by the disappearence of some columns in the training and validation sets after fine-tuning. It wasn't until I saw <a href="https://twitter.com/GuggerSylvain?s=20">Sylvain Gugger's</a> tutorial on <a href="https://github.com/huggingface/notebooks/blob/master/examples/question_answering.ipynb">question answering</a> that I realised this is by design!  Indeed, as noted in the <a href="https://huggingface.co/transformers/main_classes/trainer.html?highlight=trainer#id1">docs</a>{% fn 1 %} for the <code>train_dataset</code> and <code>eval_dataset</code> arguments of the <code>Trainer</code>:</p>
<blockquote><p>If it is an <code>datasets.Dataset</code>, columns not accepted by the <code>model.forward()</code> method are automatically removed.</p>
</blockquote>
<p>A simple one-liner to restore the missing columns is the following:</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<div class="highlight"><pre><span></span><span class="n">dataset</span><span class="o">.</span><span class="n">set_format</span><span class="p">(</span><span class="nb">type</span><span class="o">=</span><span class="n">dataset</span><span class="o">.</span><span class="n">format</span><span class="p">[</span><span class="s2">&quot;type&quot;</span><span class="p">],</span> <span class="n">columns</span><span class="o">=</span><span class="nb">list</span><span class="p">(</span><span class="n">dataset</span><span class="o">.</span><span class="n">features</span><span class="o">.</span><span class="n">keys</span><span class="p">()))</span>
</pre></div>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>To understand <em>why</em> this works, we can peek inside the relevant <code>Trainer</code> code</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="o">??</span>Trainer._remove_unused_columns
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="output_area">



<div class="output_text output_subarea ">
<pre><span class="ansi-red-fg">Signature:</span>
Trainer<span class="ansi-blue-fg">.</span>_remove_unused_columns<span class="ansi-blue-fg">(</span>
    self<span class="ansi-blue-fg">,</span>
    dataset<span class="ansi-blue-fg">:</span><span class="ansi-blue-fg">&#39;datasets.Dataset&#39;</span><span class="ansi-blue-fg">,</span>
    description<span class="ansi-blue-fg">:</span>Union<span class="ansi-blue-fg">[</span>str<span class="ansi-blue-fg">,</span> NoneType<span class="ansi-blue-fg">]</span><span class="ansi-blue-fg">=</span><span class="ansi-green-fg">None</span><span class="ansi-blue-fg">,</span>
<span class="ansi-blue-fg">)</span>
<span class="ansi-red-fg">Docstring:</span> &lt;no docstring&gt;
<span class="ansi-red-fg">Source:</span>
    <span class="ansi-green-fg">def</span> _remove_unused_columns<span class="ansi-blue-fg">(</span>self<span class="ansi-blue-fg">,</span> dataset<span class="ansi-blue-fg">:</span> <span class="ansi-blue-fg">&#34;datasets.Dataset&#34;</span><span class="ansi-blue-fg">,</span> description<span class="ansi-blue-fg">:</span> Optional<span class="ansi-blue-fg">[</span>str<span class="ansi-blue-fg">]</span> <span class="ansi-blue-fg">=</span> <span class="ansi-green-fg">None</span><span class="ansi-blue-fg">)</span><span class="ansi-blue-fg">:</span>
        <span class="ansi-green-fg">if</span> <span class="ansi-green-fg">not</span> self<span class="ansi-blue-fg">.</span>args<span class="ansi-blue-fg">.</span>remove_unused_columns<span class="ansi-blue-fg">:</span>
            <span class="ansi-green-fg">return</span>
        <span class="ansi-red-fg"># Inspect model forward signature to keep only the arguments it accepts.</span>
        signature <span class="ansi-blue-fg">=</span> inspect<span class="ansi-blue-fg">.</span>signature<span class="ansi-blue-fg">(</span>self<span class="ansi-blue-fg">.</span>model<span class="ansi-blue-fg">.</span>forward<span class="ansi-blue-fg">)</span>
        signature_columns <span class="ansi-blue-fg">=</span> list<span class="ansi-blue-fg">(</span>signature<span class="ansi-blue-fg">.</span>parameters<span class="ansi-blue-fg">.</span>keys<span class="ansi-blue-fg">(</span><span class="ansi-blue-fg">)</span><span class="ansi-blue-fg">)</span>
        <span class="ansi-red-fg"># Labels may be named label or label_ids, the default data collator handles that.</span>
        signature_columns <span class="ansi-blue-fg">+=</span> <span class="ansi-blue-fg">[</span><span class="ansi-blue-fg">&#34;label&#34;</span><span class="ansi-blue-fg">,</span> <span class="ansi-blue-fg">&#34;label_ids&#34;</span><span class="ansi-blue-fg">]</span>
        columns <span class="ansi-blue-fg">=</span> <span class="ansi-blue-fg">[</span>k <span class="ansi-green-fg">for</span> k <span class="ansi-green-fg">in</span> signature_columns <span class="ansi-green-fg">if</span> k <span class="ansi-green-fg">in</span> dataset<span class="ansi-blue-fg">.</span>column_names<span class="ansi-blue-fg">]</span>
        ignored_columns <span class="ansi-blue-fg">=</span> list<span class="ansi-blue-fg">(</span>set<span class="ansi-blue-fg">(</span>dataset<span class="ansi-blue-fg">.</span>column_names<span class="ansi-blue-fg">)</span> <span class="ansi-blue-fg">-</span> set<span class="ansi-blue-fg">(</span>signature_columns<span class="ansi-blue-fg">)</span><span class="ansi-blue-fg">)</span>
        dset_description <span class="ansi-blue-fg">=</span> <span class="ansi-blue-fg">&#34;&#34;</span> <span class="ansi-green-fg">if</span> description <span class="ansi-green-fg">is</span> <span class="ansi-green-fg">None</span> <span class="ansi-green-fg">else</span> <span class="ansi-blue-fg">f&#34;in the {description} set &#34;</span>
        logger<span class="ansi-blue-fg">.</span>info<span class="ansi-blue-fg">(</span>
            <span class="ansi-blue-fg">f&#34;The following columns {dset_description}don&#39;t have a corresponding argument in `{self.model.__class__.__name__}.forward` and have been ignored: {&#39;, &#39;.join(ignored_columns)}.&#34;</span>
        <span class="ansi-blue-fg">)</span>
        dataset<span class="ansi-blue-fg">.</span>set_format<span class="ansi-blue-fg">(</span>type<span class="ansi-blue-fg">=</span>dataset<span class="ansi-blue-fg">.</span>format<span class="ansi-blue-fg">[</span><span class="ansi-blue-fg">&#34;type&#34;</span><span class="ansi-blue-fg">]</span><span class="ansi-blue-fg">,</span> columns<span class="ansi-blue-fg">=</span>columns<span class="ansi-blue-fg">)</span>
<span class="ansi-red-fg">File:</span>      /usr/local/lib/python3.6/dist-packages/transformers/trainer.py
<span class="ansi-red-fg">Type:</span>      function
</pre>
</div>

</div>

</div>
</div>

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>and see that we're effectively undoing the final <code>dataset.set_format()</code> operation.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="A-simple-example">A simple example<a class="anchor-link" href="#A-simple-example"> </a></h2>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>To see this in action, let's grab 1,000 examples from the COLA dataset:</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">datasets</span> <span class="kn">import</span> <span class="n">load_dataset</span>

<span class="n">cola</span> <span class="o">=</span> <span class="n">load_dataset</span><span class="p">(</span><span class="s1">&#39;glue&#39;</span><span class="p">,</span> <span class="s1">&#39;cola&#39;</span><span class="p">,</span> <span class="n">split</span><span class="o">=</span><span class="s1">&#39;train[:1000]&#39;</span><span class="p">)</span>
<span class="n">cola</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="output_area">



<div class="output_text output_subarea output_execute_result">
<pre>Dataset({
    features: [&#39;sentence&#39;, &#39;label&#39;, &#39;idx&#39;],
    num_rows: 1000
})</pre>
</div>

</div>

</div>
</div>

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Here we can see that each split has three <code>Dataset.features</code>: <code>sentence</code>, <code>label</code>, and <code>idx</code>. By inspecting the <code>Dataset.format</code> attribute</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">cola</span><span class="o">.</span><span class="n">format</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="output_area">



<div class="output_text output_subarea output_execute_result">
<pre>{&#39;type&#39;: None,
 &#39;format_kwargs&#39;: {},
 &#39;columns&#39;: [&#39;idx&#39;, &#39;label&#39;, &#39;sentence&#39;],
 &#39;output_all_columns&#39;: False}</pre>
</div>

</div>

</div>
</div>

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>we also see that the <code>type</code> is <code>None</code>. Next, let's load a pretrained model and its corresponding tokenizer:</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">import</span> <span class="nn">torch</span>
<span class="kn">from</span> <span class="nn">transformers</span> <span class="kn">import</span> <span class="n">AutoTokenizer</span><span class="p">,</span> <span class="n">AutoModelForSequenceClassification</span>

<span class="n">num_labels</span> <span class="o">=</span> <span class="mi">2</span>
<span class="n">model_name</span> <span class="o">=</span> <span class="s1">&#39;distilbert-base-uncased&#39;</span>
<span class="n">device</span> <span class="o">=</span> <span class="n">torch</span><span class="o">.</span><span class="n">device</span><span class="p">(</span><span class="s2">&quot;cuda&quot;</span> <span class="k">if</span> <span class="n">torch</span><span class="o">.</span><span class="n">cuda</span><span class="o">.</span><span class="n">is_available</span><span class="p">()</span> <span class="k">else</span> <span class="s2">&quot;cpu&quot;</span><span class="p">)</span>

<span class="n">tokenizer</span> <span class="o">=</span> <span class="n">AutoTokenizer</span><span class="o">.</span><span class="n">from_pretrained</span><span class="p">(</span><span class="n">model_name</span><span class="p">)</span>
<span class="n">model</span> <span class="o">=</span> <span class="p">(</span><span class="n">AutoModelForSequenceClassification</span>
         <span class="o">.</span><span class="n">from_pretrained</span><span class="p">(</span><span class="n">model_name</span><span class="p">,</span> <span class="n">num_labels</span><span class="o">=</span><span class="n">num_labels</span><span class="p">)</span>
         <span class="o">.</span><span class="n">to</span><span class="p">(</span><span class="n">device</span><span class="p">))</span>
</pre></div>

    </div>
</div>
</div>

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Before fine-tuning the model, we need to tokenize and encode the dataset, so let's do that with a simple <code>Dataset.map</code> operation:</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="k">def</span> <span class="nf">tokenize_and_encode</span><span class="p">(</span><span class="n">batch</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">tokenizer</span><span class="p">(</span><span class="n">batch</span><span class="p">[</span><span class="s1">&#39;sentence&#39;</span><span class="p">],</span> <span class="n">truncation</span><span class="o">=</span><span class="kc">True</span><span class="p">)</span>

<span class="n">cola_enc</span> <span class="o">=</span> <span class="n">cola</span><span class="o">.</span><span class="n">map</span><span class="p">(</span><span class="n">tokenize_and_encode</span><span class="p">,</span> <span class="n">batched</span><span class="o">=</span><span class="kc">True</span><span class="p">)</span>
<span class="n">cola_enc</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="output_area">



<div class="output_text output_subarea output_execute_result">
<pre>Dataset({
    features: [&#39;attention_mask&#39;, &#39;idx&#39;, &#39;input_ids&#39;, &#39;label&#39;, &#39;sentence&#39;],
    num_rows: 1000
})</pre>
</div>

</div>

</div>
</div>

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Note that the encoding process has added two new <code>Dataset.features</code> to our dataset: <code>attention_mask</code> and <code>input_ids</code>. Since we don't care about evaluation, let's create a minimal trainer and fine-tune the model for one epoch:</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">transformers</span> <span class="kn">import</span> <span class="n">TrainingArguments</span><span class="p">,</span> <span class="n">Trainer</span>

<span class="n">batch_size</span> <span class="o">=</span> <span class="mi">16</span>
<span class="n">logging_steps</span> <span class="o">=</span> <span class="nb">len</span><span class="p">(</span><span class="n">cola_enc</span><span class="p">)</span> <span class="o">//</span> <span class="n">batch_size</span>

<span class="n">training_args</span> <span class="o">=</span> <span class="n">TrainingArguments</span><span class="p">(</span>
    <span class="n">output_dir</span><span class="o">=</span><span class="s2">&quot;results&quot;</span><span class="p">,</span>
    <span class="n">num_train_epochs</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span>
    <span class="n">per_device_train_batch_size</span><span class="o">=</span><span class="n">batch_size</span><span class="p">,</span>
    <span class="n">disable_tqdm</span><span class="o">=</span><span class="kc">False</span><span class="p">,</span>
    <span class="n">logging_steps</span><span class="o">=</span><span class="n">logging_steps</span><span class="p">)</span>

<span class="n">trainer</span> <span class="o">=</span> <span class="n">Trainer</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="n">model</span><span class="p">,</span>
    <span class="n">args</span><span class="o">=</span><span class="n">training_args</span><span class="p">,</span>
    <span class="n">train_dataset</span><span class="o">=</span><span class="n">cola_enc</span><span class="p">,</span>
    <span class="n">tokenizer</span><span class="o">=</span><span class="n">tokenizer</span><span class="p">)</span>

<span class="n">trainer</span><span class="o">.</span><span class="n">train</span><span class="p">();</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="output_area">


<div class="output_html rendered_html output_subarea ">

    <div>
        <style>
            /* Turns off some styling */
            progress {
                /* gets rid of default border in Firefox and Opera. */
                border: none;
                /* Needs to be in here for Safari polyfill so background images work as expected. */
                background-size: auto;
            }
        </style>

      <progress value='63' max='63' style='width:300px; height:20px; vertical-align: middle;'></progress>
      [63/63 00:03, Epoch 1/1]
    </div>
    <table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>Step</th>
      <th>Training Loss</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>62</td>
      <td>0.630255</td>
    </tr>
  </tbody>
</table><p>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>By inspecting one of the training examples</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">cola_enc</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="output_area">



<div class="output_text output_subarea output_execute_result">
<pre>{&#39;attention_mask&#39;: [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
 &#39;input_ids&#39;: [101,
  2256,
  2814,
  2180,
  1005,
  1056,
  4965,
  2023,
  4106,
  1010,
  2292,
  2894,
  1996,
  2279,
  2028,
  2057,
  16599,
  1012,
  102],
 &#39;label&#39;: 1}</pre>
</div>

</div>

</div>
</div>

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>it seems that we've lost our <code>sentence</code> and <code>idx</code> columns! However, by inspecting the <code>features</code> attribute</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">cola_enc</span><span class="o">.</span><span class="n">features</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="output_area">



<div class="output_text output_subarea output_execute_result">
<pre>{&#39;attention_mask&#39;: Sequence(feature=Value(dtype=&#39;int64&#39;, id=None), length=-1, id=None),
 &#39;idx&#39;: Value(dtype=&#39;int32&#39;, id=None),
 &#39;input_ids&#39;: Sequence(feature=Value(dtype=&#39;int64&#39;, id=None), length=-1, id=None),
 &#39;label&#39;: ClassLabel(num_classes=2, names=[&#39;unacceptable&#39;, &#39;acceptable&#39;], names_file=None, id=None),
 &#39;sentence&#39;: Value(dtype=&#39;string&#39;, id=None)}</pre>
</div>

</div>

</div>
</div>

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>we see that they are still present in the dataset. Applying our one-liner to restore them gives the desired result:</p>

</div>
</div>
</div>
    {% raw %}

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">cola_enc</span><span class="o">.</span><span class="n">set_format</span><span class="p">(</span><span class="nb">type</span><span class="o">=</span><span class="n">cola_enc</span><span class="o">.</span><span class="n">format</span><span class="p">[</span><span class="s2">&quot;type&quot;</span><span class="p">],</span> <span class="n">columns</span><span class="o">=</span><span class="nb">list</span><span class="p">(</span><span class="n">cola_enc</span><span class="o">.</span><span class="n">features</span><span class="o">.</span><span class="n">keys</span><span class="p">()))</span>
<span class="n">cola_enc</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="output_area">



<div class="output_text output_subarea output_execute_result">
<pre>{&#39;attention_mask&#39;: [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
 &#39;idx&#39;: 0,
 &#39;input_ids&#39;: [101,
  2256,
  2814,
  2180,
  1005,
  1056,
  4965,
  2023,
  4106,
  1010,
  2292,
  2894,
  1996,
  2279,
  2028,
  2057,
  16599,
  1012,
  102],
 &#39;label&#39;: 1,
 &#39;sentence&#39;: &#34;Our friends won&#39;t buy this analysis, let alone the next one we propose.&#34;}</pre>
</div>

</div>

</div>
</div>

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>{{ "Proof positive that I only read documentation after some threshold of confusion." | fndetail: 1 }}</p>

</div>
</div>
</div>


