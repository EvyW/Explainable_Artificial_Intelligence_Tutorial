---


---

<h1 id="explainable-artificial-intelligence">Explainable Artificial Intelligence</h1>
<p>Explainable AI (XAI) are techniques that attempt to explain the decisions taken by “black box” models in a way that is easy to understand by humans with the purpose of increasing the trust in our prediction models.  In this way we can identify whether the model has the “right” reasons  to entail its decisions.</p>
<h1 id="outline">Outline</h1>
<ol>
<li>Motivation</li>
<li>Why do we want to explain AI?</li>
<li>Approaches to explain AI (Conceptualization, examples, advantages and disadvantages)<br>
3.1. Global surrogates models<br>
3.2. Partial Dependence Plots<br>
3.3. Individual Conditional Expectation Plots<br>
3.4. Feature interaction<br>
3.5. Feature importance<br>
3.6. Shapley Values<br>
3.7. LIME<br>
3.8.Anchors</li>
<li>Use-cases</li>
</ol>
<h1 id="motivation">Motivation</h1>
<p>A real case:</p>
<ul>
<li>Microsoft Research developed a “good” system based on a neural network that detects whether a patient is going to die from pneumonia.</li>
<li>The idea was to predict whether the patient is at high or low risk. If the patient is at high risk, it must be interned in the hospital, if s/he is at low risk, s/he is sent home.</li>
<li>Just before it was deployed an error was discovered. The model believed that when people have asthma the risk of dying of pneumonia lowers. However, asthma actually increases the risk.</li>
<li>Humans could detect that this does not make sense,</li>
<li>Since patients who are admitted with asthma were treated more aggressively in the past, the model ended up believing that those patients are at lower risk because they had previously an intervention.</li>
<li>Can you imagine if they would had put the system into production? The patient would not be examined, he/she would be sent home and could possibly die.</li>
<li>The project was shutted down.  We need to understand better black box algorithms and the reasons on how decisions are taken</li>
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

