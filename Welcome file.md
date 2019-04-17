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
<li>Goal: classify according to 3 classes: (1) very likely to be admitted, (2) likely to be admitted, (3) unlikely to be admitted</li>
</ul>
<h2 id="partial-dependence-plot-pdp">Partial Dependence Plot (PDP)</h2>
<h3 id="conceptualization">Conceptualization</h3>
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
<h3 id="demo"><strong>Demo</strong></h3>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># Required libraries</span>
library<span class="token punctuation">(</span>data.table<span class="token punctuation">)</span>
library<span class="token punctuation">(</span>randomForest<span class="token punctuation">)</span>
library<span class="token punctuation">(</span>rpart<span class="token punctuation">)</span>

<span class="token comment"># Read data</span>
myData <span class="token operator">=</span> fread<span class="token punctuation">(</span><span class="token string">"data_train.csv"</span><span class="token punctuation">,</span> sep <span class="token operator">=</span> <span class="token string">";"</span><span class="token punctuation">)</span> <span class="token comment">#training data</span>
testData <span class="token operator">=</span> fread<span class="token punctuation">(</span><span class="token string">"data_test.csv"</span><span class="token punctuation">,</span> sep <span class="token operator">=</span><span class="token string">";"</span><span class="token punctuation">)</span> <span class="token comment">#test data</span>

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
<p>Create a PDP for the class: <em>very likely</em> (class “1”)  to be admitted for the Master Program</p>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># Create a PDP  </span>
library<span class="token punctuation">(</span>iml<span class="token punctuation">)</span> <span class="token comment">#library for interpretable machine learning</span>
library<span class="token punctuation">(</span>tidyverse<span class="token punctuation">)</span>

X <span class="token operator">&lt;-</span> myData <span class="token percent-operator operator">%&gt;%</span>
  select<span class="token punctuation">(</span><span class="token operator">-</span>ID<span class="token punctuation">,</span> <span class="token operator">-</span>ChanceOfAdmit<span class="token punctuation">)</span> <span class="token percent-operator operator">%&gt;%</span> <span class="token comment">#Remove the ID column and the </span>
  as.data.frame<span class="token punctuation">(</span><span class="token punctuation">)</span>

<span class="token comment"># generate the predictor</span>
predictor_pdp <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>rf_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">,</span> class <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token comment"># set the explanation for class "1"</span>
<span class="token comment"># generate the PDP function</span>
pdp <span class="token operator">=</span> FeatureEffect<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_pdp<span class="token punctuation">,</span> feature <span class="token operator">=</span> <span class="token string">"CGPA"</span><span class="token punctuation">,</span> method <span class="token operator">=</span> <span class="token string">"pdp"</span><span class="token punctuation">)</span> <span class="token comment">#set the explanation for the "CGPA" feature</span>
<span class="token comment"># plot</span>
pdp<span class="token operator">$</span>plot<span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token comment"># plot the results</span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/2J2DJgT8dF9Af1NV1qoueaRea3tiy3rcTh3LbCCB1R5UeOxnrHEDEqbvMg32xivyAEqjFW-YXX_U=s900" alt="enter image description here"><br>
From the figure we can interpret that  CGPA grades greater than 8.4 start to predict that a person has a very favorable chance to be accepted for the master program. However, grades greater than 9.3 no longer impact the predictions.<br>
Now we compare with a PDP for the class: <em>likely</em> (class “2”)  to be admitted for the Master Program</p>
<p><img src="https://lh3.googleusercontent.com/zr0hqHcgIZdh6upv4hyQ_ijjbk-jl-wCfuUdjPy-jcnbYDfnP7zET9G1Ehx6jMe3hdBKuvtoB9YD=s900" alt="enter image description here"><br>
This figure tells us that the model tends to predict “class 2” when the grades are in between ~7.4 and ~8.3, which makes sence, because the plot for “class 1” told us that the model tends to predict “class 1” when the grades are greater than 8.</p>
<h3 id="advantages-and-disadvantages"><strong>Advantages and disadvantages</strong></h3>
<p>Advantages:</p>
<ul>
<li>Easy to  compute</li>
<li>Highly  visual</li>
<li>Easy to  understand (intuitive)</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li>As PDPs plots  are  plots  of  averages, they  tend  to  obscure individual differences.</li>
<li>The maximum number of features that a PDP can display is two (plus one for the average prediction), because we can not visualize more than 3 dimensions.</li>
<li>The most important problem with PDPs is that they assume that the features are not correlated, when this might not be true. However, pdps can work well when the relations between the features of the PDP and the other features are weak.</li>
<li>Due to the permutation, we could be possibly creating data points that are not according to the reality, especially if the features are correlated.</li>
</ul>
<h2 id="individual-conditional-expectation-ice">Individual Conditional Expectation (ICE)</h2>
<h3 id="conceptualization-1"><strong>Conceptualization</strong></h3>
<ul>
<li>As it was claimed that partial dependent plots might obscure the complexity of the model relationship since it disoplay the average predictions, individual conditional expectation plots (ICEs) where introduced.</li>
<li>The major idea of ICE is then to disaggregate the average plot by displaying the estimated relationships for each observation. So instead of only plotting out the average predictions, ICE displays all individual lines. So I allow us to drill down to the level of individuals observations (Goldstein et al. 2017).</li>
<li>From the past example then a total of 3 lines per observation will be displayed.</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/ir8GJHrMvxT7g3h9BjVEyJeLKYIc7MVwr3opk_kdrjlNfJcx83O7zjBzzlC7kAKj-fvnmRz3GCg2=s300" alt="enter image description here"></p>
<h3 id="demo-1"><strong>Demo</strong></h3>
<p>Create an ICE plot for the class: <em>very likely</em> (class “1”) to be admitted for the Master Program</p>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># generate the predictor </span>
predictor_ice <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>rf_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">,</span> class <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token comment"># set the explanation for class "1"</span>
<span class="token comment"># generate ice function</span>
ice <span class="token operator">=</span> FeatureEffect<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_ice<span class="token punctuation">,</span> feature <span class="token operator">=</span> <span class="token string">"CGPA"</span><span class="token punctuation">,</span> method <span class="token operator">=</span> <span class="token string">"ice"</span><span class="token punctuation">)</span> <span class="token comment"># set the explanation for feature CGPA</span>
<span class="token comment"># plot</span>
ice<span class="token operator">$</span>plot<span class="token punctuation">(</span><span class="token punctuation">)</span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/mmT6VEP67BmgEs_s46wEs7aBuG-xxMkRnIFcrnONzuJsjEEz1AlVTOmNaWwbY0KHuz3Phn0-GAuv=s900" alt="enter image description here">Different to the PDP, all the lines are showed in the plot. What we can notice is that most of the lines follow the same pattern that the PDP (which shows the average). For all the cases a CGPA score greater than 8.4 makes the model to predict “very likely”. As the majority of lines are very similar to the PDP, then we could use only the PDP as this seems to be a good summary.<br>
If we would get an ICE plot where the patterns differ greatly among each other, then a PDP is probably not a good representation of the true patterns.</p>
<h3 id="advantages-and-disadvantages-1"><strong>Advantages and disadvantages</strong></h3>
<p>Advantages:</p>
<ul>
<li>Easy to  compute</li>
<li>Highly  visual</li>
<li>ICEs are easier to interpret than partial dependence plots. A line shows the prediction for one observation when we vary the feature of interest.</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li>Similarly to PDPs,  due to the permutation, we could be possibly creating data points that are not according to the reality, especially if the features are correlated.</li>
<li>If many curves are plotted, it becomes cluttered and difficult to read. A possible solution is to cluster lines, add some transparency, or conside only a sample.</li>
</ul>
<h2 id="feature-interaction">Feature interaction</h2>
<h3 id="conceptualization-2"><strong>Conceptualization</strong></h3>
<ul>
<li>This is a global approach that explains the interaction between features. Usually the interaction between one feature and all the others, and between two features and all the others are of interest.</li>
<li>This approach states that a prediction cannot be represented as the sum of the effect of every feature, since the effect of one feature is (possibly) influenced by another feature.</li>
<li>The interaction between two variables is understood as the change in the prediction that is additional to the change that each feature by it self will provoke (individual feature effect).</li>
<li>For example, in the prediction of the number of rented bikes in a model with two features “weather” and “month”, if the effect of “weather-good” is 900 and the effect of “month-July” is 800, and the prediction of an specific day with good weather in July is  2800 bikes, then the interaction between the features is: 2800 – (900 + 800) = 1100.</li>
<li>The feature/s effect is computed via partial dependence functions (marginal effect that a feature have on the predicted outcome, see partial dependence plots)</li>
</ul>
<p><strong>Toy example: predicting NBA shot success</strong></p>
<p><img src="https://lh3.googleusercontent.com/q-HPTaGOLJNGrwgq_FOEj8UiV0pl7NZdMnYVZnGoa_PGCiYqbLaTLoP_jEGWlJqldiChAgL7xZWB=s1000" alt="enter image description here"><br>
We can evaluate the interaction between two features, one feature and the rest, two features and the rest, three features and the rest, etc. However, the most interesting cases are usually:</p>
<ol>
<li>Two-way interaction: to what extent two features interact with each other.</li>
<li>Total interaction: to what extend a feature interacts with all the other features</li>
</ol>
<p>Two way interaction:</p>
<ol>
<li>Compute the decomposed partial dependence function (PDF): If two features don´t interact  we can break down the partial dependence function as follows.
<ul>
<li>Partial dependence function = PDF of feature 1 + PDF of feature 2</li>
<li>Formally: <span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>P</mi><mi>D</mi><msub><mi>P</mi><mi>j</mi></msub><mi>k</mi><mo>(</mo><msub><mi>x</mi><mi>j</mi></msub><mo separator="true">,</mo><msub><mi>x</mi><mi>k</mi></msub><mo>)</mo><mo>=</mo><mi>P</mi><msub><mi>D</mi><mi>j</mi></msub><mo>(</mo><msub><mi>x</mi><mi>j</mi></msub><mo>)</mo><mo>+</mo><mi>P</mi><msub><mi>D</mi><mi>k</mi></msub><mo>(</mo><msub><mi>x</mi><mi>k</mi></msub><mo>)</mo></mrow><annotation encoding="application/x-tex"> PDP_jk(x_j,x_k) = PD_j(x_j)+PD_k(x_k)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.03611em; vertical-align: -0.286108em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="mord"><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.13889em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span style="margin-right: 0.03148em;" class="mord mathit">k</span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.03611em; vertical-align: -0.286108em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span></span></span></span></span></span></li>
<li>It means that features j and k are additive</li>
</ul>
</li>
<li>Measure the difference between the observed PD function and the decomposed function:
<ul>
<li>Interaction Strength Statistic = Observed PDF – Decomposed PDF (no interactions)</li>
<li>Formally this is called H-statistic: <span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>f</mi><mi>o</mi><mi>r</mi><mi>m</mi><mi>u</mi><mi>l</mi><mi>a</mi><mo>!</mo><mo>!</mo></mrow><annotation encoding="application/x-tex">formula!! </annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.88888em; vertical-align: -0.19444em;"></span><span style="margin-right: 0.10764em;" class="mord mathit">f</span><span class="mord mathit">o</span><span style="margin-right: 0.02778em;" class="mord mathit">r</span><span class="mord mathit">m</span><span class="mord mathit">u</span><span style="margin-right: 0.01968em;" class="mord mathit">l</span><span class="mord mathit">a</span><span class="mclose">!</span><span class="mclose">!</span></span></span></span></span></span></li>
</ul>
</li>
</ol>
<ul>
<li>The statistic is 0 if there is no interaction at all, and 1 if there is full interaction, meaning that if the interaction is 1 then the effect of the prediction is achieved only throughout the interaction. If the features interact, then they are non-additive.</li>
<li>Note that the H-statistic is expensive to compute because it iterates over all data points. At each point the partial dependence has to be computed, with is done with all n data points.<br>
<img src="https://lh3.googleusercontent.com/m4_oK-5CiST4_h468SkW8_C-WF0JIrW4s2EzPc9eZ1gEP7lY7sCtQSF2v6tDBLKGL1XCo0wQ4a-H=s900" alt="enter image description here"><br>
Total interaction:</li>
</ul>
<ol>
<li>Compute the decomposed PD function (in this case it is the full prediction function): If a feature has no interaction with any of the other features, the full prediction function can be expressed as following.
<ul>
<li>Full prediction function= PDF of feature j + PDF of all other features except j</li>
<li>Formally: <span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mover accent="true"><mi>f</mi><mo>^</mo></mover><mo>(</mo><mi>x</mi><mo>)</mo><mo>=</mo><mi>P</mi><msub><mi>D</mi><mi>j</mi></msub><mo>(</mo><msub><mi>x</mi><mi>j</mi></msub><mo>)</mo><mo>+</mo><mi>P</mi><msub><mi>D</mi><mo>−</mo></msub><mi>j</mi><mo>(</mo><msub><mi>x</mi><mo>−</mo></msub><mi>j</mi><mo>)</mo></mrow><annotation encoding="application/x-tex"> \hat f(x) = PD_j(x_j)+PD_-j(x_-j)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.20788em; vertical-align: -0.25em;"></span><span class="mord accent"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.95788em;"><span class="" style="top: -3em;"><span class="pstrut" style="height: 3em;"></span><span style="margin-right: 0.10764em;" class="mord mathit">f</span></span><span class="" style="top: -3.26344em;"><span class="pstrut" style="height: 3em;"></span><span class="accent-body" style="left: -0.08333em;">^</span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.19444em;"><span class=""></span></span></span></span></span><span class="mopen">(</span><span class="mord mathit">x</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.03611em; vertical-align: -0.286108em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.258331em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mbin mtight">−</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.208331em;"><span class=""></span></span></span></span></span></span><span style="margin-right: 0.05724em;" class="mord mathit">j</span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.258331em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mbin mtight">−</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.208331em;"><span class=""></span></span></span></span></span></span><span style="margin-right: 0.05724em;" class="mord mathit">j</span><span class="mclose">)</span></span></span></span></span></span></li>
<li>This expresses the full prediction without interaction between the feature j and all the other features</li>
</ul>
</li>
<li>Measure the difference between the observed PD function and the decomposed function:
<ul>
<li>Interaction Strength Statistic = Observed full prediction – Decomposed PDF (no interactions)</li>
<li>Formally: <span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>f</mi><mi>o</mi><mi>r</mi><mi>m</mi><mi>u</mi><mi>l</mi><mi>a</mi></mrow><annotation encoding="application/x-tex"> formula </annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.88888em; vertical-align: -0.19444em;"></span><span style="margin-right: 0.10764em;" class="mord mathit">f</span><span class="mord mathit">o</span><span style="margin-right: 0.02778em;" class="mord mathit">r</span><span class="mord mathit">m</span><span class="mord mathit">u</span><span style="margin-right: 0.01968em;" class="mord mathit">l</span><span class="mord mathit">a</span></span></span></span></span></span></li>
</ul>
</li>
</ol>
<h3 id="demo-2"><strong>Demo</strong></h3>
<p>Create a feature interaction plot of type “total interaction” (interaction of one feature with all the others)</p>
<pre class=" language-r"><code class="prism  language-r">predictor_inter <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>rf_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">,</span> class <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token comment">#set the explanation for class "1"</span>
interactions <span class="token operator">&lt;-</span> Interaction<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_inter<span class="token punctuation">)</span>
<span class="token comment"># plot</span>
interactions<span class="token operator">$</span>plot<span class="token punctuation">(</span><span class="token punctuation">)</span> 
</code></pre>
<p><img src="https://lh3.googleusercontent.com/K3HhqNZZm3VutN-YyU4YZqfmMP4HSZ57NcMPc5j_N99E2EJdXzAKlEkJuNYhAVlBrPwxgFgt9fS_=s900" alt="enter image description here">The H-statistic plot shows that indeed the features interact with each other.  The CGPA grade shows to have the highest interaction with all the other features. However, in general the interaction with all the features is relatively low.</p>
<h3 id="advantages-and-disadvantages-2">Advantages and disadvantages</h3>
<h2 id="local-interpretable-model-agnostic-explanations-lime">Local interpretable model-agnostic explanations (LIME)</h2>
<h3 id="conceptualization-3">Conceptualization</h3>
<ul>
<li>The technique attempts to understand the model by perturbing the input of data samples and understanding how the predictions change.</li>
<li>LIME provides local model interpretability. LIME modifies a single data sample by tweaking the feature values and observes the resulting impact on the output.</li>
<li>It tries to answer: which variables caused the prediction?</li>
<li>The output of LIME is a list of explanations, reflecting the contribution of each feature to the prediction of a data sample.</li>
<li>An explanation is created by approximating the underlying model locally by an interpretable one. Interpretable models are e.g. linear models with strong regularisation, decision tree’s, etc. The interpretable models are trained on small perturbations of the original instance and should only provide a good local approximation.</li>
<li>The ‘dataset’ is created by e.g. adding noise to continuous features, removing words or hiding parts of the image. By only approximating the black-box locally (in the neighborhood of the data sample) the task is significantly simplified.</li>
<li>Behind the workings of lime lies the (big) assumption that every complex model is linear on a local scale. While this is not justified in the paper it is not difficult to convince yourself that this is generally sound — you usually expect two very similar observations to behave predictably even in a complex model.</li>
<li>LIME then asserts that it is possible to fit a simple model around a single observation that will mimic how the global model behaves at that locality (in the parameter space just around my data point). The simple model can then be used to explain the predictions of the more complex model locally.<br>
<img src="https://lh3.googleusercontent.com/SWNEjIyAQ_kI3Ie2uECO5eKDcqIG07lk7V-9sCOQyEpWGkVJt3GmO2xsqFEwCMXO9QgjRyM0vgnc=s900" alt="enter image description here"></li>
</ul>
<p><strong>How does it work in general?</strong></p>
<ol>
<li>For each prediction to explain, permute the observation n times.</li>
<li>Let the complex model predict the outcome of all permuted observations.</li>
<li>Calculate the distance from all permutations to the original observation.</li>
<li>Convert the distance into a similarity score.</li>
<li>Select m features best describing the complex model outcome from the permuted data.</li>
<li>Fit a simple model to the permuted data with the m features (from the permuted data weighted by its similarity to the original observation)(how does it add this weight?).</li>
<li>Extract the feature weights from the simple model and use these as explanations for the complex models local behavior.<br>
<img src="https://lh3.googleusercontent.com/kCUsIS2ECrWIp21bw1GjhyAlBy-CqSHrewDf3i5pPAlB4_IGtFHtF6-m6NaVXPk_4CiHvm5LWiP9=s900" alt="enter image description here"></li>
</ol>
<p><strong>LIME in detail</strong></p>
<p>Permutation:  LIME depends on the type of input data. Currently two types of inputs are supported: tabular and text</p>
<ul>
<li>Tabular  data: when  tabular  data, the permutations are dependent on the training set. During the creation of the explainer the statistics for each variable are extracted and permutations are then sampled from the variable distributions (no treally the variable distribution, but more like from the variable range). This means that permutations are in fact independent from the explained variable (because when permuting (creating the new data points, relationship among variables is not considered, so realtionships are destroyed) making the similarity computation even more important as this is the only thing establishing the locality of the analysis.</li>
<li>Which statistics are extracted?: we compute the mean and std, and discretize it into quartiles (what are quartiles for?). If it is categorical data then  we compute the frequency of each value.</li>
<li>Text data: When the outcome of text predictions are to be explained the permutations are performed by randomly removing words from the original observation. Depending on whether the model uses word location or not, words occurring multiple times will be removed one-by-one or as a whole.</li>
<li>What are the statistics for?
<ul>
<li>To scale the data, so that we can meaningfully compute distances when the attributes are not on the same scale</li>
<li>To sample perturbed instances - which we do by sampling from a Normal(0,1), multiplying by the std and adding back the mean. (they assume a normal distribution)</li>
</ul>
</li>
</ul>
<p>Computing the similarity score: it is calculated differently according to whether it is for tabular or textual data</p>
<ul>
<li>Textual: For text data the cosine similarity measure is used, which is the standard in text analysis</li>
<li>Tabular: For tabular data an optimal solution will depend on the type of input data
<ol>
<li>Categorical: categorical features will be recoded based on whether or not they are equal to the observation (maybe 1 if they are equal and 0 if not)</li>
<li>Continuous: (if continuous features are binned these features will be recoded based on whether they are in the same bin as the observation.) The distance to the original observation is then calculated based on a user-chosen distance measure (euclidean by default), and converted to a similarity using an exponential kernel of a user defined width (defaults to 0.75 times the square root of the number of features)</li>
</ol>
</li>
</ul>
<p>Selecting features: LIME implements a range of different feature selection approaches that the user is free to choose from (e.g. “forward selection”, “lasso”, etc.).</p>
<ul>
<li>The user must select the number of features. The number must strike a balance between the complexity of the model and the simplicity of the explanation, some suggests to keep it below 10 features.</li>
</ul>
<h3 id="demo-3">Demo</h3>
<p>This time we are going to generate explanations for predictions resulting from a SVM</p>
<pre class=" language-r"><code class="prism  language-r">library<span class="token punctuation">(</span>e1071<span class="token punctuation">)</span> <span class="token comment">#to generate the svm model</span>

<span class="token comment">#create model and show summary</span>
svm_model <span class="token operator">&lt;-</span> svm<span class="token punctuation">(</span>x<span class="token punctuation">,</span>y<span class="token punctuation">)</span>
summary<span class="token punctuation">(</span>svm_model<span class="token punctuation">)</span>

<span class="token comment">#run prediction and show execution time</span>
pred <span class="token operator">&lt;-</span> predict<span class="token punctuation">(</span>svm_model<span class="token punctuation">,</span> x<span class="token punctuation">)</span>
system.time<span class="token punctuation">(</span>pred <span class="token operator">&lt;-</span> predict<span class="token punctuation">(</span>svm_model<span class="token punctuation">,</span>x<span class="token punctuation">)</span><span class="token punctuation">)</span>

<span class="token comment"># load predictions as data.table</span>
predictionsTable <span class="token operator">=</span> as.data.frame<span class="token punctuation">(</span>table<span class="token punctuation">(</span>pred<span class="token punctuation">,</span>y<span class="token punctuation">)</span><span class="token punctuation">)</span> 
write.csv<span class="token punctuation">(</span>predictionsTable<span class="token punctuation">,</span> file <span class="token operator">=</span> <span class="token string">"predictions.csv"</span><span class="token punctuation">)</span> <span class="token comment"># Write CSV in R</span>
predictionsTable <span class="token operator">=</span> fread<span class="token punctuation">(</span><span class="token string">"predictions.csv"</span><span class="token punctuation">)</span> <span class="token comment"># load as data.table</span>

<span class="token comment"># select predictions</span>
predictionsTable <span class="token operator">=</span> predictionsTable<span class="token punctuation">[</span><span class="token number">1</span><span class="token operator">:</span><span class="token number">450</span><span class="token punctuation">,</span><span class="token punctuation">]</span>

<span class="token comment"># change to integers </span>
predictionsTable <span class="token operator">=</span> predictionsTable<span class="token punctuation">[</span><span class="token punctuation">,</span> pred <span class="token operator">:</span><span class="token operator">=</span> ifelse<span class="token punctuation">(</span>predictionsTable<span class="token operator">$</span>pred <span class="token operator">&gt;=</span> <span class="token number">2.5</span><span class="token punctuation">,</span> <span class="token number">3</span><span class="token punctuation">,</span> 
                                                               ifelse<span class="token punctuation">(</span>myData<span class="token operator">$</span>ChanceOfAdmit <span class="token operator">&gt;=</span> <span class="token number">1.5</span><span class="token punctuation">,</span> <span class="token number">2</span><span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
<span class="token comment"># confusion matrix</span>
predictionVector <span class="token operator">=</span> predictionsTable<span class="token punctuation">[</span><span class="token punctuation">,</span> pred<span class="token punctuation">]</span>
table<span class="token punctuation">(</span>predictionVector<span class="token punctuation">,</span>y<span class="token punctuation">)</span>



</code></pre>
<pre class=" language-r"><code class="prism  language-r">library<span class="token punctuation">(</span>lime<span class="token punctuation">)</span> 
library<span class="token punctuation">(</span>e1071<span class="token punctuation">)</span> <span class="token comment">#to generate the svm model</span>

<span class="token comment"># select 1 observation from the test set to explain with lime</span>
localObs <span class="token operator">=</span> read.csv<span class="token punctuation">(</span><span class="token string">"data_test.csv"</span><span class="token punctuation">,</span> sep <span class="token operator">=</span> <span class="token string">";"</span><span class="token punctuation">)</span>
localObs <span class="token operator">=</span> subset<span class="token punctuation">(</span>localObs<span class="token punctuation">,</span> select<span class="token operator">=</span><span class="token operator">-</span>c<span class="token punctuation">(</span>ID<span class="token punctuation">,</span> ChanceOfAdmit<span class="token punctuation">)</span><span class="token punctuation">)</span>
localObs <span class="token operator">=</span> localObs<span class="token punctuation">[</span><span class="token number">1</span><span class="token operator">:</span><span class="token number">5</span><span class="token punctuation">,</span><span class="token punctuation">]</span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/CwEqlvqvSWO9DZCMFhHoi-G5wCOuGKaWyUDDwO3VwebhEg7t8IniVi5OwnQ-5S_kF_vLFBzIiNM5=s900" alt="enter image description here"></p>

