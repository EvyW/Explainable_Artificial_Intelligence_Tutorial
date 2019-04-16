---


---

<h1 id="explainable-artificial-intelligence">Explainable Artificial Intelligence</h1>
<p>Explainable AI (XAI) are techniques that attempt to explain the decisions taken by “black box” models in a way that is easy to understand by humans with the purpose of increasing the trust in our prediction models.  In this way we can identify whether the model has the “right” reasons  to entail its decisions.</p>
<h1 id="outline">Outline</h1>
<ol>
<li>Motivation</li>
<li>Why do we want to explain AI?</li>
<li>Approaches to explain AI (Conceptualization, examples, advantages and disadvantages)
<ol start="3">
<li>Global surrogates models</li>
<li>Partial Dependence Plots</li>
<li>Individual Conditional Expectation Plots</li>
<li>Feature interaction</li>
<li>Feature importance</li>
<li>Shapley Values</li>
<li>LIME<br>
10.Anchors</li>
</ol>
</li>
<li>Use-cases</li>
</ol>
<h1 id="motivation">Motivation</h1>
<ul>
<li>Microsoft Research developed a “good” system based on a neural network that detects whether a patient is going to die from pneumonia.</li>
<li>The idea was to predict whether the patient is at high or low risk. If the patient is at high risk, it must be interned in the hospital, if s/he is at low risk, s/he is sent home.</li>
<li>Just before it was deployed an error was discovered. The model believe that when people have asthma the risk of dying of pneumonia lowers. Humans could detect that this does not make sense, asthma increases the risk.</li>
<li>Since patients who are admitted with asthma were treated more aggressively, those patients ended up having a lower risk because there was previously an intervention.</li>
<li>Can you imagine if the have would put the system into production? The patient would not be intervened and would be sent home with a probability of dying.</li>
<li>The project was shutted down. We need to understand better black box<br>
algorithms and the reasons on how decisions are taken</li>
</ul>
<h1 id="why-do-we-want-to-explain-ai">Why do we want to explain AI?</h1>
<ul>
<li>Without a good understanding of the method, you are very likely to<br>
base your assumptions on falsehoods</li>
<li>model interpretation is important for data science to gain the trust from managements and customers.</li>
<li>So, how can we make sure we have trained or are provided with a model that we can actually use in production? (entonces: safer for putting it into production)</li>
</ul>
<h1 id="approaches-to-explain-ai">Approaches to explain AI</h1>
<ol>
<li>Model-specific approaches: is designed to provide explanations of an especific black box algorithm.</li>
<li>Model-agnostic approaches: provide explanations regardless of the black box algorithm. We can apply them to any model.</li>
<li>Local approaches: provides explanations for understanding the prediction of a single instance.</li>
<li>Global approaches: provides explanations for understanding the general behavior of the model.</li>
</ol>
<p><strong>Model-specific vs. model agnostic approaches</strong></p>
<p>As model-agnostic approaches are decoupled from the algorithm that was used, they have the advantage of being modular and therefore they can be used with any model.</p>
<h2 id="dataset">Dataset</h2>
<p>Note that the dataset used for the running example has the following characteristics:</p>
<ul>
<li>Dataset: Graduate Admissions</li>
<li>Description: the dataset contains several parameters which are considered important during the application for Masters Programs in order to be admitted.</li>
<li>Attributes:
<ul>
<li>GRE Scores ( out of 340 )</li>
<li>TOEFL Scores ( out of 120 )</li>
<li>University Rating ( out of 5 )</li>
<li>Statement of Purpose</li>
<li>Letter of Recommendation Strength ( out of 5 )</li>
<li>Undergraduate GPA ( out of 10 )</li>
<li>Research experience ( either 0 or 1 )</li>
<li>Chance of Admit ( ranging from 0 to 1 )</li>
</ul>
</li>
<li>Goal: classify according to 3 categories: (1) very high chance to be admitted, (2) chance to be admitted, (3) no chance to be admitted</li>
</ul>
<h2 id="partial-dependence-plot-pdp">Partial Dependence Plot (PDP)</h2>
<ul>
<li>The partial dependence plot (PDP) shows the average marginal effect that one or two features (S) have on the predicted outcome of a machine learning model.</li>
<li>Partial dependence works by marginalizing the machine learning model output over the distribution of the all the features (excluding the ones that we want to plot), so that the function shows the relationship between the features in set S we are interested in and the predicted outcome. <strong>In simple words, it shows how my model will respond or changes if I increase or decrease x. How model inputs affect the model’s predictions.</strong>  -</li>
<li>The plot shows the relationship between the features that we want to display and the predicted outcome. PDPs can show whether the relationship between the target and a feature is linear,  monotonous or more complex.</li>
</ul>
<p><strong>Toy example: predicting NBA shot success</strong></p>
<p><img src="https://lh3.googleusercontent.com/wF5Ra2ogzaN8T-jc5zJCMO3spH5q7mWOvznAO02kxYMW0UL9CYwmkYMfHbpHIC-a5eLXHh4skNUg=s900" alt="enter image description here"><br>
We compute <em>y</em> by generating estimations on different values of the feature that we want to plot.</p>
<p><img src="https://lh3.googleusercontent.com/kAjcx1bElarY9VeU9xN2EjRVu_qzsBlOOD3ihKLp-O7odYns_nscqgULO6KO5XOrV7Lc8ImoxK_S=s900" alt=""><br>
<strong>Other use cases</strong><br>
Partial dependence plots can be used to compare models</p>
<p><img src="https://lh3.googleusercontent.com/zPoolaI3Oxrbq_xd-y8XV9SXUAiiUGpw79syR3zJ4MBY_5Vax3lFM8SuVI8Hj82CzsE4kEwGf1i5=s500" alt="enter image description here"></p>
<ul>
<li>Note that only random forest predicts a significant drop off in success when shot are taken from around 30 feet or more. Which makes more sense. Due to the long distance, the shot is probably less likely to succeed.</li>
</ul>
<p>Overlaying partial dependence of different models can help to choose models that are not only accurate, but also make intuitive sense.</p>
<p><strong>Demo</strong></p>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># Required libraries</span>
library<span class="token punctuation">(</span>lime<span class="token punctuation">)</span> 
library<span class="token punctuation">(</span>data.table<span class="token punctuation">)</span>
library<span class="token punctuation">(</span>e1071<span class="token punctuation">)</span>
library<span class="token punctuation">(</span>randomForest<span class="token punctuation">)</span>
library<span class="token punctuation">(</span>rpart<span class="token punctuation">)</span>
library<span class="token punctuation">(</span>iml<span class="token punctuation">)</span>

<span class="token comment"># Read data</span>
myData <span class="token operator">=</span> fread<span class="token punctuation">(</span><span class="token string">"data_train.csv"</span><span class="token punctuation">,</span> sep <span class="token operator">=</span> <span class="token string">";"</span><span class="token punctuation">)</span>
testData <span class="token operator">=</span> fread<span class="token punctuation">(</span><span class="token string">"data_test.csv"</span><span class="token punctuation">,</span> sep <span class="token operator">=</span><span class="token string">";"</span><span class="token punctuation">)</span>

summary<span class="token punctuation">(</span>myData<span class="token punctuation">)</span>
</code></pre>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment">##       ID           GREScore       TOEFLScore    UniversityRating StatementOfPurpose</span>
<span class="token comment">## Min.   :  1.0   Min.   :290.0   Min.   : 92.0   Min.   :1.00     Min.   :1.000     </span>
<span class="token comment">## 1st Qu.:113.2   1st Qu.:309.0   1st Qu.:103.0   1st Qu.:2.00     1st Qu.:2.500     </span>
<span class="token comment">## Median :225.5   Median :317.0   Median :107.0   Median :3.00     Median :3.500     </span>
<span class="token comment">## Mean   :225.5   Mean   :316.7   Mean   :107.3   Mean   :3.08     Mean   :3.373     </span>
<span class="token comment">## 3rd Qu.:337.8   3rd Qu.:325.0   3rd Qu.:112.0   3rd Qu.:4.00     3rd Qu.:4.000     </span>
<span class="token comment">## Max.   :450.0   Max.   :340.0   Max.   :120.0   Max.   :5.00     Max.   :5.000     </span>
<span class="token comment">## RecommendationLetter      CGPA          Research      ChanceOfAdmit     </span>
<span class="token comment">## Min.   :1.000        Min.   :6.800   Min.   :0.0000   Length:450        </span>
<span class="token comment">## 1st Qu.:3.000        1st Qu.:8.133   1st Qu.:0.0000   Class :character  </span>
<span class="token comment">## Median :3.500        Median :8.570   Median :1.0000   Mode  :character  </span>
<span class="token comment">## Mean   :3.478        Mean   :8.585   Mean   :0.5556                     </span>
<span class="token comment">## 3rd Qu.:4.000        3rd Qu.:9.060   3rd Qu.:1.0000                     </span>
<span class="token comment">## Max.   :5.000        Max.   :9.920   Max.   :1.0000 </span>
</code></pre>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># Build a random forest</span>
myDataNoIndex <span class="token operator">=</span> myData<span class="token punctuation">[</span><span class="token punctuation">,</span><span class="token number">2</span><span class="token operator">:</span><span class="token number">9</span><span class="token punctuation">]</span> <span class="token comment">#Exclude ID column</span>
rf_model <span class="token operator">=</span> randomForest<span class="token punctuation">(</span>ChanceOfAdmit <span class="token operator">~</span> .<span class="token punctuation">,</span> data <span class="token operator">=</span> myDataNoIndex<span class="token punctuation">,</span> mtry <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">,</span> importance <span class="token operator">=</span> <span class="token boolean">TRUE</span><span class="token punctuation">)</span>  
rf_model <span class="token comment"># confusion matrix</span>
</code></pre>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment">##        OOB estimate of  error rate: 15.33%</span>
<span class="token comment">## Confusion matrix:</span>
<span class="token comment">##    1   2 3 class.error</span>
<span class="token comment">## 1 117  24 0  0.17021277</span>
<span class="token comment">## 2  16 258 0  0.05839416</span>
<span class="token comment">## 3   0  29 6  0.82857143</span>
</code></pre>
<p>Create a PDP for the class: <em>high probability</em> (class “1”)  of being admitted for the Master Program</p>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># Create a PDP  </span>
library<span class="token punctuation">(</span>tidyverse<span class="token punctuation">)</span>

X <span class="token operator">&lt;-</span> myData <span class="token percent-operator operator">%&gt;%</span>
  select<span class="token punctuation">(</span><span class="token operator">-</span>ID<span class="token punctuation">,</span> <span class="token operator">-</span>ChanceOfAdmit<span class="token punctuation">)</span> <span class="token percent-operator operator">%&gt;%</span> <span class="token comment">#Remove the ID column and the </span>
  as.data.frame<span class="token punctuation">(</span><span class="token punctuation">)</span>

predictor_pdp <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>rf_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">,</span> class <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token comment"># explanation for class "1"</span>
pdp <span class="token operator">=</span> FeatureEffect<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_pdp<span class="token punctuation">,</span> feature <span class="token operator">=</span> <span class="token string">"CGPA"</span><span class="token punctuation">,</span> method <span class="token operator">=</span> <span class="token string">"pdp"</span><span class="token punctuation">)</span> <span class="token comment">#explanation for the "CGPA" feature</span>
pdp<span class="token operator">$</span>plot<span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token comment"># plot the results</span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/2J2DJgT8dF9Af1NV1qoueaRea3tiy3rcTh3LbCCB1R5UeOxnrHEDEqbvMg32xivyAEqjFW-YXX_U=s900" alt="enter image description here"></p>
<pre class=" language-r"><code class="prism  language-r">
</code></pre>
<h2 id="open-a-file">Open a file</h2>
<p>You can open a file from <strong>Google Drive</strong>, <strong>Dropbox</strong> or <strong>GitHub</strong> by opening the <strong>Synchronize</strong> sub-menu and clicking <strong>Open from</strong>. Once opened in the workspace, any modification in the file will be automatically synced.</p>
<h2 id="save-a-file">Save a file</h2>
<p>You can save any file of the workspace to <strong>Google Drive</strong>, <strong>Dropbox</strong> or <strong>GitHub</strong> by opening the <strong>Synchronize</strong> sub-menu and clicking <strong>Save on</strong>. Even if a file in the workspace is already synced, you can save it to another location. StackEdit can sync one file with multiple locations and accounts.</p>
<h2 id="synchronize-a-file">Synchronize a file</h2>
<p>Once your file is linked to a synchronized location, StackEdit will periodically synchronize it by downloading/uploading any modification. A merge will be performed if necessary and conflicts will be resolved.</p>
<p>If you just have modified your file and you want to force syncing, click the <strong>Synchronize now</strong> button in the navigation bar.</p>
<blockquote>
<p><strong>Note:</strong> The <strong>Synchronize now</strong> button is disabled if you have no file to synchronize.</p>
</blockquote>
<h2 id="manage-file-synchronization">Manage file synchronization</h2>
<p>Since one file can be synced with multiple locations, you can list and manage synchronized locations by clicking <strong>File synchronization</strong> in the <strong>Synchronize</strong> sub-menu. This allows you to list and remove synchronized locations that are linked to your file.</p>
<h1 id="publication">Publication</h1>
<p>Publishing in StackEdit makes it simple for you to publish online your files. Once you’re happy with a file, you can publish it to different hosting platforms like <strong>Blogger</strong>, <strong>Dropbox</strong>, <strong>Gist</strong>, <strong>GitHub</strong>, <strong>Google Drive</strong>, <strong>WordPress</strong> and <strong>Zendesk</strong>. With <a href="http://handlebarsjs.com/">Handlebars templates</a>, you have full control over what you export.</p>
<blockquote>
<p>Before starting to publish, you must link an account in the <strong>Publish</strong> sub-menu.</p>
</blockquote>
<h2 id="publish-a-file">Publish a File</h2>
<p>You can publish your file by opening the <strong>Publish</strong> sub-menu and by clicking <strong>Publish to</strong>. For some locations, you can choose between the following formats:</p>
<ul>
<li>Markdown: publish the Markdown text on a website that can interpret it (<strong>GitHub</strong> for instance),</li>
<li>HTML: publish the file converted to HTML via a Handlebars template (on a blog for example).</li>
</ul>
<h2 id="update-a-publication">Update a publication</h2>
<p>After publishing, StackEdit keeps your file linked to that publication which makes it easy for you to re-publish it. Once you have modified your file and you want to update your publication, click on the <strong>Publish now</strong> button in the navigation bar.</p>
<blockquote>
<p><strong>Note:</strong> The <strong>Publish now</strong> button is disabled if your file has not been published yet.</p>
</blockquote>
<h2 id="manage-file-publication">Manage file publication</h2>
<p>Since one file can be published to multiple locations, you can list and manage publish locations by clicking <strong>File publication</strong> in the <strong>Publish</strong> sub-menu. This allows you to list and remove publication locations that are linked to your file.</p>
<h1 id="markdown-extensions">Markdown extensions</h1>
<p>StackEdit extends the standard Markdown syntax by adding extra <strong>Markdown extensions</strong>, providing you with some nice features.</p>
<blockquote>
<p><strong>ProTip:</strong> You can disable any <strong>Markdown extension</strong> in the <strong>File properties</strong> dialog.</p>
</blockquote>
<h2 id="smartypants">SmartyPants</h2>
<p>SmartyPants converts ASCII punctuation characters into “smart” typographic punctuation HTML entities. For example:</p>

<table>
<thead>
<tr>
<th></th>
<th>ASCII</th>
<th>HTML</th>
</tr>
</thead>
<tbody>
<tr>
<td>Single backticks</td>
<td><code>'Isn't this fun?'</code></td>
<td>‘Isn’t this fun?’</td>
</tr>
<tr>
<td>Quotes</td>
<td><code>"Isn't this fun?"</code></td>
<td>“Isn’t this fun?”</td>
</tr>
<tr>
<td>Dashes</td>
<td><code>-- is en-dash, --- is em-dash</code></td>
<td>– is en-dash, — is em-dash</td>
</tr>
</tbody>
</table><h2 id="katex">KaTeX</h2>
<p>You can render LaTeX mathematical expressions using <a href="https://khan.github.io/KaTeX/">KaTeX</a>:</p>
<p>The <em>Gamma function</em> satisfying <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi mathvariant="normal">Γ</mi><mo>(</mo><mi>n</mi><mo>)</mo><mo>=</mo><mo>(</mo><mi>n</mi><mo>−</mo><mn>1</mn><mo>)</mo><mo>!</mo><mspace width="1em"></mspace><mi mathvariant="normal">∀</mi><mi>n</mi><mo>∈</mo><mi mathvariant="double-struck">N</mi></mrow><annotation encoding="application/x-tex">\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord">Γ</span><span class="mopen">(</span><span class="mord mathit">n</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mopen">(</span><span class="mord mathit">n</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord">1</span><span class="mclose">)</span><span class="mclose">!</span><span class="mspace" style="margin-right: 1em;"></span><span class="mord">∀</span><span class="mord mathit">n</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">∈</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.68889em; vertical-align: 0em;"></span><span class="mord mathbb">N</span></span></span></span></span> is via the Euler integral</p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi mathvariant="normal">Γ</mi><mo>(</mo><mi>z</mi><mo>)</mo><mo>=</mo><msubsup><mo>∫</mo><mn>0</mn><mi mathvariant="normal">∞</mi></msubsup><msup><mi>t</mi><mrow><mi>z</mi><mo>−</mo><mn>1</mn></mrow></msup><msup><mi>e</mi><mrow><mo>−</mo><mi>t</mi></mrow></msup><mi>d</mi><mi>t</mi>&amp;ThinSpace;<mi mathvariant="normal">.</mi></mrow><annotation encoding="application/x-tex">
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord">Γ</span><span class="mopen">(</span><span style="margin-right: 0.04398em;" class="mord mathit">z</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 2.32624em; vertical-align: -0.91195em;"></span><span class="mop"><span style="margin-right: 0.44445em; position: relative; top: -0.001125em;" class="mop op-symbol large-op">∫</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.41429em;"><span class="" style="top: -1.78805em; margin-left: -0.44445em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">0</span></span></span><span class="" style="top: -3.8129em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">∞</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.91195em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathit">t</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.864108em;"><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span style="margin-right: 0.04398em;" class="mord mathit mtight">z</span><span class="mbin mtight">−</span><span class="mord mtight">1</span></span></span></span></span></span></span></span></span><span class="mord"><span class="mord mathit">e</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.843556em;"><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">−</span><span class="mord mathit mtight">t</span></span></span></span></span></span></span></span></span><span class="mord mathit">d</span><span class="mord mathit">t</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord">.</span></span></span></span></span></span></p>
<blockquote>
<p>You can find more information about <strong>LaTeX</strong> mathematical expressions <a href="http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference">here</a>.</p>
</blockquote>
<h2 id="uml-diagrams">UML diagrams</h2>
<p>You can render UML diagrams using <a href="https://mermaidjs.github.io/">Mermaid</a>. For example, this will produce a sequence diagram:</p>
<div class="mermaid"><svg xmlns="http://www.w3.org/2000/svg" id="mermaid-svg-IgOgutdzWbw3a2cg" height="100%" width="100%" style="max-width:750px;" viewBox="-50 -10 750 471.4000015258789"><g></g><g><line id="actor12" x1="75" y1="5" x2="75" y2="460.4000015258789" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="0" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="75" y="32.5" style="text-anchor: middle;" dominant-baseline="central" alignment-baseline="central" class="actor"><tspan x="75" dy="0">Alice</tspan></text></g><g><line id="actor13" x1="275" y1="5" x2="275" y2="460.4000015258789" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="200" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="275" y="32.5" style="text-anchor: middle;" dominant-baseline="central" alignment-baseline="central" class="actor"><tspan x="275" dy="0">Bob</tspan></text></g><g><line id="actor14" x1="475" y1="5" x2="475" y2="460.4000015258789" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="400" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="475" y="32.5" style="text-anchor: middle;" dominant-baseline="central" alignment-baseline="central" class="actor"><tspan x="475" dy="0">John</tspan></text></g><defs><marker id="arrowhead" refX="5" refY="2" markerWidth="6" markerHeight="4" orient="auto"><path d="M 0,0 V 4 L6,2 Z"></path></marker></defs><defs><marker id="crosshead" markerWidth="15" markerHeight="8" orient="auto" refX="16" refY="4"><path fill="black" stroke="#000000" style="stroke-dasharray: 0, 0;" stroke-width="1px" d="M 9,2 V 6 L16,4 Z"></path><path fill="none" stroke="#000000" style="stroke-dasharray: 0, 0;" stroke-width="1px" d="M 0,1 L 6,7 M 6,1 L 0,7"></path></marker></defs><g><text x="175" y="93" style="text-anchor: middle;" class="messageText">Hello Bob, how are you?</text><line x1="75" y1="100" x2="275" y2="100" class="messageLine0" stroke-width="2" stroke="black" style="fill: none;" marker-end="url(#arrowhead)"></line></g><g><text x="375" y="128" style="text-anchor: middle;" class="messageText">How about you John?</text><line x1="275" y1="135" x2="475" y2="135" style="stroke-dasharray: 3, 3; fill: none;" class="messageLine1" stroke-width="2" stroke="black" marker-end="url(#arrowhead)"></line></g><g><text x="175" y="163" style="text-anchor: middle;" class="messageText">I am good thanks!</text><line x1="275" y1="170" x2="75" y2="170" style="stroke-dasharray: 3, 3; fill: none;" class="messageLine1" stroke-width="2" stroke="black" marker-end="url(#crosshead)"></line></g><g><text x="375" y="198" style="text-anchor: middle;" class="messageText">I am good thanks!</text><line x1="275" y1="205" x2="475" y2="205" class="messageLine0" stroke-width="2" stroke="black" style="fill: none;" marker-end="url(#crosshead)"></line></g><g><rect x="500" y="215" fill="#EDF2AE" stroke="#666" width="150" height="90.4000015258789" rx="0" ry="0" class="note"></rect><text x="496" y="239" fill="black" class="noteText"><tspan x="516" fill="black">Bob thinks a long</tspan></text><text x="496" y="256.6000003814697" fill="black" class="noteText"><tspan x="516" fill="black">long time, so long</tspan></text><text x="496" y="274.20000076293945" fill="black" class="noteText"><tspan x="516" fill="black">that the text does</tspan></text><text x="496" y="291.8000011444092" fill="black" class="noteText"><tspan x="516" fill="black">not fit on a row.</tspan></text></g><g><text x="175" y="333.4000015258789" style="text-anchor: middle;" class="messageText">Checking with John...</text><line x1="275" y1="340.4000015258789" x2="75" y2="340.4000015258789" style="stroke-dasharray: 3, 3; fill: none;" class="messageLine1" stroke-width="2" stroke="black"></line></g><g><text x="275" y="368.4000015258789" style="text-anchor: middle;" class="messageText">Yes... John, how are you?</text><line x1="75" y1="375.4000015258789" x2="475" y2="375.4000015258789" class="messageLine0" stroke-width="2" stroke="black" style="fill: none;"></line></g><g><rect x="0" y="395.4000015258789" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="75" y="427.9000015258789" style="text-anchor: middle;" dominant-baseline="central" alignment-baseline="central" class="actor"><tspan x="75" dy="0">Alice</tspan></text></g><g><rect x="200" y="395.4000015258789" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="275" y="427.9000015258789" style="text-anchor: middle;" dominant-baseline="central" alignment-baseline="central" class="actor"><tspan x="275" dy="0">Bob</tspan></text></g><g><rect x="400" y="395.4000015258789" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="475" y="427.9000015258789" style="text-anchor: middle;" dominant-baseline="central" alignment-baseline="central" class="actor"><tspan x="475" dy="0">John</tspan></text></g></svg></div>
<p>And this will produce a flow chart:</p>
<div class="mermaid"><svg xmlns="http://www.w3.org/2000/svg" id="mermaid-svg-mkw89RXpevuUqLO6" width="100%" style="max-width: 501.69165802001953px;" viewBox="0 0 501.69165802001953 174.18331909179688"><g transform="translate(-12, -12)"><g class="output"><g class="clusters"></g><g class="edgePaths"><g class="edgePath" style="opacity: 1;"><path class="path" d="M120.50296188568976,79.42082977294922L179.7750015258789,50.73332977294922L254.90833282470703,50.73332977294922" marker-end="url(#arrowhead33)" style="fill:none"></path><defs><marker id="arrowhead33" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M120.50296188568976,126.13748931884766L179.7750015258789,154.82498931884766L235.06666564941406,154.82498931884766" marker-end="url(#arrowhead34)" style="fill:none"></path><defs><marker id="arrowhead34" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M316.37499237060547,50.73332977294922L361.21665954589844,50.73332977294922L409.4466793773049,80.54914047743144" marker-end="url(#arrowhead35)" style="fill:none"></path><defs><marker id="arrowhead35" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M336.21665954589844,154.82498931884766L361.21665954589844,154.82498931884766L409.4466784320204,126.0091791949599" marker-end="url(#arrowhead36)" style="fill:none"></path><defs><marker id="arrowhead36" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g></g><g class="edgeLabels"><g class="edgeLabel" style="opacity: 1;" transform="translate(179.7750015258789,50.73332977294922)"><g transform="translate(-30.291664123535156,-13.358329772949219)" class="label"><foreignObject width="60.58332824707031" height="26.716659545898438"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Link text</span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g></g><g class="nodes"><g class="node" style="opacity: 1;" id="A" transform="translate(72.24166870117188,102.77915954589844)"><rect rx="0" ry="0" x="-52.241668701171875" y="-23.35832977294922" width="104.48333740234375" height="46.71665954589844"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-42.241668701171875,-13.358329772949219)"><foreignObject width="84.48333740234375" height="26.716659545898438"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Square Rect</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="B" transform="translate(285.64166259765625,50.73332977294922)"><circle x="-30.73332977294922" y="-23.35832977294922" r="30.73332977294922"></circle><g class="label" transform="translate(0,0)"><g transform="translate(-20.73332977294922,-13.358329772949219)"><foreignObject width="41.46665954589844" height="26.716659545898438"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Circle</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="C" transform="translate(285.64166259765625,154.82498931884766)"><rect rx="5" ry="5" x="-50.57499694824219" y="-23.35832977294922" width="101.14999389648438" height="46.71665954589844"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-40.57499694824219,-13.358329772949219)"><foreignObject width="81.14999389648438" height="26.716659545898438"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Round Rect</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="D" transform="translate(445.954158782959,102.77915954589844)"><polygon points="59.737500000000004,0 119.47500000000001,-59.737500000000004 59.737500000000004,-119.47500000000001 0,-59.737500000000004" rx="5" ry="5" transform="translate(-59.737500000000004,59.737500000000004)"></polygon><g class="label" transform="translate(0,0)"><g transform="translate(-33.01667022705078,-13.358329772949219)"><foreignObject width="66.03334045410156" height="26.716659545898438"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Rhombus</div></foreignObject></g></g></g></g></g></g></svg></div>

