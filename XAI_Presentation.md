---


---

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
<p>A real case: Amazon recruiting tool</p>
<ul>
<li>The retailer Amazon experimented with an AI recruiting tool that showed bias against women.</li>
<li>This tool was developed to assist the human resource department for classifying  job applicants according to their suitability.</li>
<li>It was revealed that the model penalized applicants with a resume containing words like female or woman.</li>
<li>Amazon adjusted the model to be neutral to gender-related terminology but decided nevertheless to shut down the project since there was no guarantee that the model did not find other ways to discriminate applicants.</li>
</ul>
<h1 id="approaches-to-explain-ai">Approaches to explain AI</h1>
<p><img src="https://lh3.googleusercontent.com/ndXZ9iX-KD42mT-5hTbKxJXGq_XCfGvhL4QbYwDX6AaqBcdTQ79ThedmA3RhFIGU39AvJfn0rJTq=s900" alt="enter image description here"></p>
<p><strong>Model-specific vs. model agnostic approaches</strong></p>
<p>As model-agnostic approaches are decoupled from the algorithm that was used, they have the advantage of being modular and therefore they can be used with any model. The rest of this document is then focused on model-agnostic approaches.</p>
<h2 id="dataset">Dataset</h2>
<p>Note that the dataset used for the running example has the following characteristics:</p>
<ul>
<li>Dataset: Graduate Admissions</li>
<li>Description: the dataset contains several parameters which are considered important during the application for Masters Programs in order to be admitted.</li>
<li>Attributes:</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/PgT3MXB9T5QWcN8P_6x618EhmkHgW6XQDyxspf2qLsnA8rm__qRN5MLHMszVAfq0xcOu1Ly1xW2N=s900" alt="enter image description here"></p>
<ul>
<li>Goal: classify according to 3 categories (1) very likely to be admitted, (2) likely to be admitted, (3) unlikely to be admitted</li>
</ul>
<h2 id="global-surrogate-models">Global surrogate models</h2>
<h3 id="conceptualization">Conceptualization</h3>
<p>A surrogate model is a glass-box model that is learned on the predictions of a black-box model to approximate its behavior.</p>
<h3 id="demo">Demo</h3>
<h3 id="advantages-and-disadvantages">Advantages and Disadvantages</h3>
<p>Advantages:</p>
<ul>
<li>The surrogate model method is flexible: any model that is interpretable can be used.</li>
<li>The approach is very intuitive and straightforward to implement.</li>
<li>Easy to explain to people not familiar with data science or machine learning.</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li>It is not clear what is the best R-squared  to be confident that the surrogate model is close enough to the black box model</li>
</ul>
<h2 id="partial-dependence-plot-pdp">Partial Dependence Plot (PDP)</h2>
<h3 id="general-idea">General idea</h3>
<ul>
<li>The partial dependence plot (PDP) shows the average effect that one or two features have on the predicted outcome of a machine learning model.</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/GwmV9eOm4eCyzyDTJygrnIFPdwNwbWjF8I3OOzFciUmAzgnSl4m0dg2Wfo48x8zfW-ug5I9Kg6hd=s500" alt="enter image description here"></p>
<p><strong>Interpretation:</strong></p>
<ul>
<li>When the distance is 10m the average probability of being - classified as a successful shot is around 0.38</li>
<li>Values lower than 10 begin to predict ”successful shot" more strongly than values greater than 10</li>
</ul>
<h3 id="demo-1"><strong>Demo</strong></h3>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># Required libraries</span>
library<span class="token punctuation">(</span>data.table<span class="token punctuation">)</span>
library<span class="token punctuation">(</span>iml<span class="token punctuation">)</span>

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
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># XGBoost model</span>

library<span class="token punctuation">(</span>xgboost<span class="token punctuation">)</span>
library<span class="token punctuation">(</span>caret<span class="token punctuation">)</span>

xgb_trcontrol <span class="token operator">=</span> trainControl<span class="token punctuation">(</span>
  method <span class="token operator">=</span> <span class="token string">"cv"</span><span class="token punctuation">,</span>
  number <span class="token operator">=</span> <span class="token number">5</span><span class="token punctuation">,</span>  
  allowParallel <span class="token operator">=</span> <span class="token boolean">TRUE</span><span class="token punctuation">,</span>
  verboseIter <span class="token operator">=</span> <span class="token boolean">FALSE</span><span class="token punctuation">,</span>
  returnData <span class="token operator">=</span> <span class="token boolean">FALSE</span><span class="token punctuation">,</span>
  classProbs <span class="token operator">=</span> <span class="token boolean">TRUE</span>
<span class="token punctuation">)</span>

xgbGrid <span class="token operator">&lt;-</span> expand.grid<span class="token punctuation">(</span>nrounds <span class="token operator">=</span> c<span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">,</span><span class="token number">200</span><span class="token punctuation">)</span><span class="token punctuation">,</span>  <span class="token comment"># this is n_estimators in the python code above</span>
                       max_depth <span class="token operator">=</span> c<span class="token punctuation">(</span><span class="token number">10</span><span class="token punctuation">,</span> <span class="token number">15</span><span class="token punctuation">,</span> <span class="token number">20</span><span class="token punctuation">,</span> <span class="token number">25</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
                       colsample_bytree <span class="token operator">=</span> seq<span class="token punctuation">(</span><span class="token number">0.5</span><span class="token punctuation">,</span> <span class="token number">0.9</span><span class="token punctuation">,</span> length.out <span class="token operator">=</span> <span class="token number">5</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
                       <span class="token comment">## The values below are default values in the sklearn-api. </span>
                       eta <span class="token operator">=</span> <span class="token number">0.1</span><span class="token punctuation">,</span>
                       gamma<span class="token operator">=</span><span class="token number">0</span><span class="token punctuation">,</span>
                       min_child_weight <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">,</span>
                       subsample <span class="token operator">=</span> <span class="token number">1</span>
<span class="token punctuation">)</span>

<span class="token comment"># first the target feature must be converted to factors</span>
myData <span class="token operator">=</span> myData<span class="token punctuation">[</span><span class="token punctuation">,</span><span class="token number">2</span><span class="token operator">:</span><span class="token number">9</span><span class="token punctuation">]</span> <span class="token comment">#Exclude ID column</span>
myData<span class="token operator">$</span>ChanceOfAdmit <span class="token operator">=</span> as.factor<span class="token punctuation">(</span>myData<span class="token operator">$</span>ChanceOfAdmit<span class="token punctuation">)</span>
levels<span class="token punctuation">(</span>myData<span class="token operator">$</span>ChanceOfAdmit<span class="token punctuation">)</span> <span class="token operator">=</span> c<span class="token punctuation">(</span><span class="token string">"very_likely"</span><span class="token punctuation">,</span> <span class="token string">"likely"</span><span class="token punctuation">,</span> <span class="token string">"unlikely"</span><span class="token punctuation">)</span>

<span class="token comment"># Build xgb model (using caret library)</span>
set.seed<span class="token punctuation">(</span><span class="token number">103</span><span class="token punctuation">)</span>  <span class="token comment"># for reproducibility</span>
xgboost_model <span class="token operator">&lt;-</span> train<span class="token punctuation">(</span>ChanceOfAdmit <span class="token operator">~</span> .<span class="token punctuation">,</span> data <span class="token operator">=</span> myData<span class="token punctuation">,</span> method <span class="token operator">=</span> <span class="token string">"xgbTree"</span><span class="token punctuation">,</span>
                  prob.model <span class="token operator">=</span> <span class="token boolean">TRUE</span><span class="token punctuation">,</span> na.action <span class="token operator">=</span> na.omit<span class="token punctuation">,</span> trControl <span class="token operator">=</span> xgb_trcontrol<span class="token punctuation">,</span>
                  tuneGrid <span class="token operator">=</span> xgbGrid<span class="token punctuation">)</span>
<span class="token comment"># predict </span>
prediction <span class="token operator">&lt;-</span> predict<span class="token punctuation">(</span>xgboost_model<span class="token punctuation">,</span> newdata <span class="token operator">=</span> myData<span class="token punctuation">[</span><span class="token punctuation">,</span> <span class="token operator">-</span>c<span class="token punctuation">(</span><span class="token string">"ChanceOfAdmit"</span><span class="token punctuation">)</span><span class="token punctuation">]</span><span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">)</span>
head<span class="token punctuation">(</span>prediction<span class="token punctuation">)</span>
</code></pre>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment">##   very_likely      likely     unlikely</span>
<span class="token comment">## 1 0.994887412 0.004491114 0.0006215358</span>
<span class="token comment">## 2 0.139787257 0.856316745 0.0038959824</span>
<span class="token comment">## 3 0.002215040 0.985944510 0.0118404422</span>
<span class="token comment">## 4 0.824476182 0.164099842 0.0114240330</span>
<span class="token comment">## 5 0.001391372 0.988758683 0.0098499591</span>
<span class="token comment">## 6 0.989802063 0.008568874 0.0016291001</span>
</code></pre>
<p>Create a PDP for the class: <em>very likely</em> (class “1”)  to be admitted for the Master Program</p>
<pre class=" language-r"><code class="prism  language-r">library<span class="token punctuation">(</span>iml<span class="token punctuation">)</span> <span class="token comment">#library for interpretable machine learning</span>
library<span class="token punctuation">(</span>tidyverse<span class="token punctuation">)</span>

X <span class="token operator">&lt;-</span> myData <span class="token percent-operator operator">%&gt;%</span>
  select<span class="token punctuation">(</span><span class="token operator">-</span>ChanceOfAdmit<span class="token punctuation">)</span> <span class="token percent-operator operator">%&gt;%</span>
  as.data.frame<span class="token punctuation">(</span><span class="token punctuation">)</span>

<span class="token comment"># generate pdp plot</span>
predictor_pdp <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>xgboost_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">)</span>
pdp <span class="token operator">=</span> FeatureEffect<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_pdp<span class="token punctuation">,</span> feature <span class="token operator">=</span> <span class="token string">"CGPA"</span><span class="token punctuation">,</span> method <span class="token operator">=</span> <span class="token string">"pdp"</span><span class="token punctuation">)</span>
pdp<span class="token operator">$</span>plot<span class="token punctuation">(</span><span class="token punctuation">)</span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/AQ9bHMDsN9JrhUAVU-FmivZUjKX1-aC9dozvPC6xbfYAxMkO5DkFCQ0ba8vjBJgKDA6XPlkYkFaD=s700" alt="enter image description here"></p>
<p>Interpretation plot “very_likely”:</p>
<ul>
<li>CGPA grades greater than 8.4 start to predict that a person has a very favorable chance to be accepted for the master program.</li>
<li>However, grades greater than 9.3 no longer increase the chance.</li>
</ul>
<p>Interpretation plot “likely”:</p>
<ul>
<li>The model tends to predict “likely” when the grades are in between ~7.4 and ~8.3, which makes sence, because the plot for “very_likely” told us that the model tends to predict “very_likely” when the grades are greater than 8.4</li>
</ul>
<p><strong>PDPs for model comparison</strong></p>
<p><img src="https://lh3.googleusercontent.com/Gnp40D29dMN095EIE8niouhYJGX-ySjVwoDhAnp2uj5fWY_DMt5xuX6JjWJdwaO_1wGTgOJQxMgf=s900" alt="enter image description here"></p>
<h3 id="pdps-in-detail">PDPs in detail</h3>
<ul>
<li>Partial dependence works by marginalizing the machine learning model output over the distribution of all the features (excluding the ones that we want to plot), so that the function shows the relationship between the features we are interested in and the predicted outcome. <strong>In simple words, it shows how my model will respond or changes if I increase or decrease x. How model inputs affect the model’s predictions.</strong></li>
<li>The plot shows the relationship between the features that we want to display and the predicted outcome. PDPs can show whether the relationship between the target and a feature is linear,  monotonous or more complex.</li>
</ul>
<p><strong>How to compute a pdp?</strong></p>
<p>Toy example: predicting NBA shot success</p>
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
<h3 id="advantages-and-disadvantages-1"><strong>Advantages and disadvantages</strong></h3>
<p>Advantages:</p>
<ul>
<li>Easy to  compute</li>
<li>Highly  visual</li>
<li>Easy to  understand (intuitive)</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li>As PDPs plots  are  plots  of  averages, they  could  o<strong>bscure individual differences</strong>.</li>
<li>The maximum number of features that a PDP can display is two (plus one for the average prediction), because we can not visualize more than 3 dimensions.</li>
<li>The most important problem with PDPs is that <strong>they assume that the features are not correlated</strong>, when this might not be true. However, pdps can work well when the relations between the features of the PDP and the other features are weak.</li>
<li><strong>Since we vary the feature of interest we could be possibly creating data points that are not according to the reality</strong>, especially if the features are correlated.</li>
</ul>
<h2 id="individual-conditional-expectation-ice">Individual Conditional Expectation (ICE)</h2>
<h3 id="general-idea-1"><strong>General idea</strong></h3>
<ul>
<li>As it was claimed that partial dependent plots might obscure the complexity of the model relationship since it display the average predictions, individual conditional expectation plots (ICEs) where introduced.</li>
</ul>
<h3 id="demo-2"><strong>Demo</strong></h3>
<p>Create an ICE plot for the class:  “very_likely” to be admitted for the Master Program</p>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># plot ice </span>
predictor_ice <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>xgboost_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">)</span>
ice <span class="token operator">=</span> FeatureEffect<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_ice<span class="token punctuation">,</span> feature <span class="token operator">=</span> <span class="token string">"CGPA"</span><span class="token punctuation">,</span> method <span class="token operator">=</span> <span class="token string">"ice"</span><span class="token punctuation">)</span>
ice<span class="token operator">$</span>plot<span class="token punctuation">(</span><span class="token punctuation">)</span>

</code></pre>
<p><img src="https://lh3.googleusercontent.com/o7T_6o_sErRA6UPKM1utGWhdZdN2u-YxoiEVL34yqQBMpMldi_-wSc3_VyAARlhKl-nr-iL1Lf79=s900" alt="enter image description here"><br>
<img src="https://lh3.googleusercontent.com/xW-dHDnoHRPOiVzUhQoCprTIGzKd5DsQVc0mUzlAXmZ0xd42bI1dl81ZwZKQm2EjeXK7GfCKzZG3=s900" alt="enter image description here"></p>
<p>Different to the PDP, all the lines are showed in the plot. What we can notice is that most of the lines follow the same pattern that the PDP (which shows the average). For all the cases a CGPA score greater than 8.4 makes the model to predict “very likely”. As the majority of lines are very similar to the PDP, then we could use only the PDP as this seems to be a good summary.<br>
If we would get an ICE plot where the patterns differ greatly among each other, then a PDP is probably not a good representation of the true patterns.</p>
<p>As opposite, the following figure represents an ice plot for cancer prediction that reveals that not all lines follow (roughly) the same pattern.</p>
<p><img src="https://lh3.googleusercontent.com/HKsiZiSKB9MOdnxrhGYNa0-u0VM5F8yMCa53n50U6u1Fk-X-xAY5pfe3mpYxqPnKbKiO5D9H1xr6=s900" alt="enter image description here"></p>
<h3 id="ice-in-detail">ICE in detail</h3>
<ul>
<li>The major idea of ice is then to disaggregate the average plot by displaying the estimated relationships for each observation.</li>
<li>From the past example (pdp example) then a total of 3 lines per observation will be displayed.</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/ir8GJHrMvxT7g3h9BjVEyJeLKYIc7MVwr3opk_kdrjlNfJcx83O7zjBzzlC7kAKj-fvnmRz3GCg2=s300" alt="enter image description here"></p>
<h3 id="advantages-and-disadvantages-2"><strong>Advantages and disadvantages</strong></h3>
<p>Advantages:</p>
<ul>
<li>Easy to  compute</li>
<li>Highly  visual</li>
<li>ICEs are easier to interpret than partial dependence plots. A line shows the prediction for one observation when we vary the feature of interest.</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li>Similarly to PDPs,  since we vary the feature, we could be <strong>possibly creating data points that are not according to the reality</strong>, especially if the features are correlated.</li>
<li>If many curves are plotted, it becomes cluttered and difficult to read. A possible solution is to cluster lines, add some transparency, or conside only a sample.</li>
</ul>
<h2 id="feature-interaction">Feature interaction</h2>
<h3 id="general-idea-2"><strong>General idea</strong></h3>
<ul>
<li>This is a global approach that explains the interaction between features. Usually the interaction between one feature and all the others.</li>
<li>This approach states that the effect of one feature (over the prediction) is possibly influenced by another feature.</li>
<li>General idea of how to compute the feature interaction:</li>
</ul>
<p><strong>Predicting NBA shot success</strong></p>
<p><img src="https://lh3.googleusercontent.com/MUca-5goRyGD9bUOtssnmX--VV80ILm0tvPZxZ3D3Kmrv4lw1mRluwLpvN4GgTsNuj4BhXUDPPA9=s900" alt=""><br>
Note that the feature/s effect is computed via partial dependence functions (marginal effect that a feature have on the predicted outcome, see partial dependence plots)</p>
<p>We can evaluate the interaction between two features, one feature and the rest, two features and the rest, three features and the rest, etc. However, the most interesting cases are usually:</p>
<ol>
<li>Two-way interaction: to what extent two features interact with each other.</li>
<li>Total interaction: to what extend a feature interacts with all the other features</li>
</ol>
<h3 id="demo-3"><strong>Demo</strong></h3>
<p>Create a feature interaction plot of one feature against all the others</p>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment"># Plot interactions</span>
predictor_inter <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>xgboost_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">)</span>
interactions <span class="token operator">&lt;-</span> Interaction<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_inter<span class="token punctuation">)</span>
interactions<span class="token operator">$</span>plot<span class="token punctuation">(</span><span class="token punctuation">)</span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/z_rw9VsW49LquKH4Ny4kdv8FltNumpP2uVUf6rnG3m03d4zs9mvrrC1kU4BcU6KTZTj3VatXTnd8=s1000" alt="enter image description here"></p>
<p>Interpretation:</p>
<ul>
<li>The plot shows that indeed the features interact with each other.</li>
<li>The CGPA grade shows to have the highest interaction with all the other features.</li>
<li>Most of the features have a “weak” interaction, below 0.25</li>
</ul>
<h3 id="feature-interaction-in-detail">Feature interaction in detail</h3>
<ul>
<li>This approach states that a prediction cannot be represented as the sum of the effect of every feature.</li>
<li>The interaction between two variables is understood as the change in the prediction that is additional to the change that each feature by it self will provoke (individual feature effect).</li>
<li>For example, in the prediction of the number of rented bikes in a model with two features “weather” and “month”, if the effect of “weather-good” is 900 and the effect of “month-July” is 800, and the prediction of an specific day with good weather in July is  2800 bikes, then the interaction between the features is: 2800 – (900 + 800) = 1100.</li>
</ul>
<p><strong>Computation of feature interactions</strong></p>
<p>Two way interaction:</p>
<ol>
<li>Compute the decomposed partial dependence function (PDF): If two features don´t interact  we can break down the partial dependence function as follows.
<ul>
<li>Partial dependence function = PDF of feature 1 + PDF of feature 2</li>
<li>Formally:<br>
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>P</mi><mi>D</mi><msub><mi>P</mi><mi>j</mi></msub><mi>k</mi><mo>(</mo><msub><mi>x</mi><mi>j</mi></msub><mo separator="true">,</mo><msub><mi>x</mi><mi>k</mi></msub><mo>)</mo><mo>=</mo><mi>P</mi><msub><mi>D</mi><mi>j</mi></msub><mo>(</mo><msub><mi>x</mi><mi>j</mi></msub><mo>)</mo><mo>+</mo><mi>P</mi><msub><mi>D</mi><mi>k</mi></msub><mo>(</mo><msub><mi>x</mi><mi>k</mi></msub><mo>)</mo></mrow><annotation encoding="application/x-tex"> PDP_jk(x_j,x_k) = PD_j(x_j)+PD_k(x_k)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.03611em; vertical-align: -0.286108em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="mord"><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.13889em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span style="margin-right: 0.03148em;" class="mord mathit">k</span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.03611em; vertical-align: -0.286108em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span></span></span></span></span></span></li>
<li>It means that features j and k are additive</li>
</ul>
</li>
<li>Measure the difference between the observed PD function and the decomposed function:
<ul>
<li>Interaction Strength Statistic = Observed PDF – Decomposed PDF (no interactions)</li>
<li>Formally this is called H-statistic:<br>
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><msubsup><mi>H</mi><mrow><mi>j</mi><mi>k</mi></mrow><mn>2</mn></msubsup><mo>=</mo><mfrac><mrow><munderover><mo>∑</mo><mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow><mi>n</mi></munderover><mo>[</mo><mi>P</mi><msub><mi>D</mi><mrow><mi>j</mi><mi>k</mi></mrow></msub><mo>(</mo><msubsup><mi>x</mi><mi>j</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msubsup><mo separator="true">,</mo><msubsup><mi>x</mi><mi>k</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msubsup><mo>)</mo><mo>−</mo><mi>P</mi><mi>D</mi><mo>(</mo><msubsup><mi>x</mi><mi>j</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msubsup><mo>)</mo><mo>−</mo><mi>P</mi><msub><mi>D</mi><mi>k</mi></msub><mo>(</mo><msubsup><mi>x</mi><mi>k</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msubsup><mo>)</mo><msup><mo>]</mo><mn>2</mn></msup></mrow><mrow><munderover><mo>∑</mo><mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow><mi>n</mi></munderover><mi>P</mi><msubsup><mi>D</mi><mrow><mi>j</mi><mi>k</mi></mrow><mn>2</mn></msubsup><mo>(</mo><msubsup><mi>x</mi><mi>j</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msubsup><mo separator="true">,</mo><msubsup><mi>x</mi><mi>k</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msubsup><mo>)</mo></mrow></mfrac></mrow><annotation encoding="application/x-tex">  H_{jk}^2 = \frac{ \sum_{i=1}^{n}  [ PD_{jk}(x_j^{(i)}, x_k^{(i)}) - PD(x_j^{(i)}) - PD_k(x_k^{(i)}) ]^2 } { \sum_{i=1}^{n} PD_{jk}^2 (x_j^{(i)}, x_k^{(i)} )} </annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.24722em; vertical-align: -0.383108em;"></span><span class="mord"><span style="margin-right: 0.08125em;" class="mord mathit">H</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.864108em;"><span class="" style="top: -2.453em; margin-left: -0.08125em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.383108em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 3.21999em; vertical-align: -1.37222em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.84777em;"><span class="" style="top: -2.11em;"><span class="pstrut" style="height: 3.0448em;"></span><span class="mord"><span class="mop"><span class="mop op-symbol small-op" style="position: relative; top: -0.000005em;">∑</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.804292em;"><span class="" style="top: -2.40029em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">i</span><span class="mrel mtight">=</span><span class="mord mtight">1</span></span></span></span><span class="" style="top: -3.2029em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">n</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.29971em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.166667em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.795908em;"><span class="" style="top: -2.39869em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="" style="top: -3.0448em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.437416em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.0448em;"><span class="" style="top: -2.42314em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span><span class="" style="top: -3.2198em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.412972em;"><span class=""></span></span></span></span></span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.0448em;"><span class="" style="top: -2.39869em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span><span class="" style="top: -3.2198em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.301308em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span></span></span><span class="" style="top: -3.2748em;"><span class="pstrut" style="height: 3.0448em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.84777em;"><span class="pstrut" style="height: 3.0448em;"></span><span class="mord"><span class="mop"><span class="mop op-symbol small-op" style="position: relative; top: -0.000005em;">∑</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.804292em;"><span class="" style="top: -2.40029em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">i</span><span class="mrel mtight">=</span><span class="mord mtight">1</span></span></span></span><span class="" style="top: -3.2029em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">n</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.29971em;"><span class=""></span></span></span></span></span></span><span class="mopen">[</span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.0448em;"><span class="" style="top: -2.42314em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span><span class="" style="top: -3.2198em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.412972em;"><span class=""></span></span></span></span></span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.0448em;"><span class="" style="top: -2.39869em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span><span class="" style="top: -3.2198em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.301308em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.0448em;"><span class="" style="top: -2.42314em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span><span class="" style="top: -3.2198em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.412972em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.0448em;"><span class="" style="top: -2.39869em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span><span class="" style="top: -3.2198em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.301308em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mclose"><span class="mclose">]</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.814108em;"><span class="" style="top: -3.063em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span></span></span></span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 1.37222em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span></span></span></span></span></li>
</ul>
</li>
</ol>
<ul>
<li>The statistic is 0 if there is no interaction at all, and 1 if there is full interaction, meaning that if the interaction is 1 then the effect of the prediction is achieved only throughout the interaction. If the features interact, then they are non-additive.</li>
<li>Note that the H-statistic is expensive to compute because it iterates over all data points, where at each point the partial dependence has to be computed, which is done with all n data points. However, if we want to reduce the computation we could select only a part of the data points.</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/m4_oK-5CiST4_h468SkW8_C-WF0JIrW4s2EzPc9eZ1gEP7lY7sCtQSF2v6tDBLKGL1XCo0wQ4a-H=s900" alt="enter image description here"><br>
Total interaction:</p>
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
<li>Formally:<br>
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><msubsup><mi>H</mi><mi>j</mi><mn>2</mn></msubsup><mo>=</mo><mfrac><mrow><munderover><mo>∑</mo><mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow><mi>n</mi></munderover><mo>[</mo><mover accent="true"><mi>f</mi><mo>^</mo></mover><mo>(</mo><msup><mi>x</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msup><mo>)</mo><mo>−</mo><mi>P</mi><mi>D</mi><mo>(</mo><msubsup><mi>x</mi><mi>j</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msubsup><mo>)</mo><mo>−</mo><mi>P</mi><msub><mi>D</mi><mrow><mo>−</mo><mi>j</mi></mrow></msub><mo>(</mo><msubsup><mi>x</mi><mrow><mo>−</mo><mi>j</mi></mrow><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msubsup><mo>)</mo><msup><mo>]</mo><mn>2</mn></msup></mrow><mrow><munderover><mo>∑</mo><mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow><mi>n</mi></munderover><mover accent="true"><mi>f</mi><mo>^</mo></mover><mo>(</mo><msup><mi>x</mi><mrow><mo>(</mo><mi>i</mi><mo>)</mo></mrow></msup><mo>)</mo></mrow></mfrac></mrow><annotation encoding="application/x-tex">  H_{j}^2 = \frac{ \sum_{i=1}^{n}  [ \hat f (x^{(i)}) - PD(x_j^{(i)}) - PD_{-j}(x_{-j}^{(i)}) ]^2 } { \sum_{i=1}^{n} \hat f (x^{(i)} )} </annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.24722em; vertical-align: -0.383108em;"></span><span class="mord"><span style="margin-right: 0.08125em;" class="mord mathit">H</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.864108em;"><span class="" style="top: -2.453em; margin-left: -0.08125em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.383108em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 2.99536em; vertical-align: -1.14759em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.84777em;"><span class="" style="top: -2.19692em;"><span class="pstrut" style="height: 3.0448em;"></span><span class="mord"><span class="mop"><span class="mop op-symbol small-op" style="position: relative; top: -0.000005em;">∑</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.804292em;"><span class="" style="top: -2.40029em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">i</span><span class="mrel mtight">=</span><span class="mord mtight">1</span></span></span></span><span class="" style="top: -3.2029em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">n</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.29971em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord accent"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.95788em;"><span class="" style="top: -3em;"><span class="pstrut" style="height: 3em;"></span><span style="margin-right: 0.10764em;" class="mord mathit">f</span></span><span class="" style="top: -3.26344em;"><span class="pstrut" style="height: 3em;"></span><span class="accent-body" style="left: -0.08333em;">^</span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.19444em;"><span class=""></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.814em;"><span class="" style="top: -2.989em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span></span></span></span></span><span class="mclose">)</span></span></span><span class="" style="top: -3.2748em;"><span class="pstrut" style="height: 3.0448em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.84777em;"><span class="pstrut" style="height: 3.0448em;"></span><span class="mord"><span class="mop"><span class="mop op-symbol small-op" style="position: relative; top: -0.000005em;">∑</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.804292em;"><span class="" style="top: -2.40029em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">i</span><span class="mrel mtight">=</span><span class="mord mtight">1</span></span></span></span><span class="" style="top: -3.2029em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">n</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.29971em;"><span class=""></span></span></span></span></span></span><span class="mopen">[</span><span class="mord accent"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.95788em;"><span class="" style="top: -3em;"><span class="pstrut" style="height: 3em;"></span><span style="margin-right: 0.10764em;" class="mord mathit">f</span></span><span class="" style="top: -3.26344em;"><span class="pstrut" style="height: 3em;"></span><span class="accent-body" style="left: -0.08333em;">^</span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.19444em;"><span class=""></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.888em;"><span class="" style="top: -3.063em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.0448em;"><span class="" style="top: -2.42314em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span><span class="" style="top: -3.2198em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.412972em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mord"><span style="margin-right: 0.02778em;" class="mord mathit">D</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">−</span><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathit">x</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.0448em;"><span class="" style="top: -2.42314em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">−</span><span style="margin-right: 0.05724em;" class="mord mathit mtight">j</span></span></span></span><span class="" style="top: -3.2198em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mopen mtight">(</span><span class="mord mathit mtight">i</span><span class="mclose mtight">)</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.412972em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span><span class="mclose"><span class="mclose">]</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.814108em;"><span class="" style="top: -3.063em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span></span></span></span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 1.14759em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span></span></span></span></span></li>
</ul>
</li>
</ol>
<h3 id="advantages-and-disadvantages-3">Advantages and disadvantages</h3>
<p>Advantages:</p>
<ul>
<li>The interactions can be detected among an arbitrary number of features.</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li>It is a computational expensive expensive algorithm (due to the computation of partial dependence functions at each point.)</li>
<li>In order to reduce the computation, a sample of points can be used. However, the results can variate significantly for every sample (as it is possible to see in the example below). Executing the algorithm several times and comparing the results to check the stability might be helpful.</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/4t0l53mr9PwKUTR-gPjDQBmFJX76XA6Y-U4k1NuwjG2zewmcsQzeQG5G4XWS8Bo0OZEHGqq7LE-Q=s1100" alt="enter image description here"></p>
<ul>
<li>It is not easy to determine what is a “strong” or “weak” interaction</li>
<li>As this technique is based on partial dependence functions, it also carries the problem of <strong>possibly creating data points that are not according to the reality</strong></li>
</ul>
<h2 id="permutation-feature-importance">Permutation Feature Importance</h2>
<h3 id="general-idea-3">General idea</h3>
<ul>
<li>It is a global explanation approach that identifies the contribution of each feature based on its accuracy.</li>
<li>A feature is important if shuffling its values increases the model error, because it would mean that the model relied on the feature for the prediction.</li>
</ul>
<h3 id="demo-4"><strong>Demo</strong></h3>
<p>Note that the loss function used for this example is “cross-entropy”</p>
<pre class=" language-r"><code class="prism  language-r">predictor_imp <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>xgboost_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> y <span class="token operator">=</span> myData<span class="token operator">$</span>ChanceOfAdmit<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">)</span>
importace_rf <span class="token operator">=</span> FeatureImp<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_imp<span class="token punctuation">,</span> loss <span class="token operator">=</span> <span class="token string">"ce"</span><span class="token punctuation">,</span> compare <span class="token operator">=</span> <span class="token string">'difference'</span><span class="token punctuation">,</span> n.repetitions <span class="token operator">=</span> <span class="token number">60</span><span class="token punctuation">)</span>
importace_rf<span class="token operator">$</span>plot<span class="token punctuation">(</span><span class="token punctuation">)</span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/vQxSCCCiw1OFOM73iHzo2Mt0EmHxK5cTdXYvkOjL5nEGUAW0qBVFnoxicjqqfN6gaGb_32uLBQqB=s900" alt="enter image description here"><br>
Interpretation:</p>
<ul>
<li>The feature with the highest importance is “CGPA”,  the increase of the error due to the permutation is around 0.24</li>
<li>The least important features are “Research”, “University”, and “Statement of purpose”</li>
</ul>
<h3 id="permutation-feature-importance-in-detail">Permutation feature importance in detail</h3>
<ul>
<li>The importance of a feature is the increase in the prediction error of the model after we permuted the feature’s values, which will break the relationship between the feature and the true outcome.</li>
<li>We measure the importance of a feature by calculating the increase in the model’s prediction error after permuting the feature. A feature is important if shuffling its values increases the model error, because it would mean that the model relied on the feature for the prediction. Then, a feature is not important if shuffling its values leaves the model error unchanged.</li>
<li>The permutation feature importance measurement was introduced originally for random forest, and based on this a new model agnostic approach was created</li>
</ul>
<p><strong>The permutation feature importance algorithm based on Fisher et al. (2018):</strong></p>
<ul>
<li>Input: Trained model f, feature matrix X, target vector y, error measure e.</li>
</ul>
<ol>
<li>Estimate the original model error (e.g. mean squared error): eorig</li>
<li>For each feature j = 1,…,p:
<ul>
<li>Generate a feature matrix by permuting feature j of the X data.</li>
<li>Estimate the error based on the predictions of the permuted data: e<sup>perm</sup></li>
<li>Calculate permutation feature importance: FI<sub>j</sub>= e<sup>perm</sup> / e<sup>orig</sup>, or alternatively FI<sub>j</sub> = e<sup>perm</sup> – e<sup>orig</sup></li>
</ul>
</li>
<li>Sort features by descending FI</li>
</ol>
<p><strong>Things to consider</strong></p>
<ul>
<li>Feature importance values are always positive values greater than 0, since the minimum increase in error is 0. A feature with an importance of 0 is interpreted as a feature that does not contribute to the model and thus we can consider to remove it.</li>
<li>To get more accurate results we can permute every feature value with all possible instances, however this can be an expensive task. Thus, the authors (Fisher et al.) suggest to split the dataset in half and exchange their feature values (of j), which is basically a permutation.</li>
<li>It is possible that a feature has a small random feature importance:
<ul>
<li>To identify such variables with only random importance, a variable random can be added to the data with values generated at random.</li>
<li>All features with an importance less than random can be considered as unimportant.</li>
</ul>
</li>
<li>Shall we get the feature importances from training data or from test data? It depends, we need to decide whether:
<ul>
<li>We want to know how much the model relies on each feature for making predictions à training data, or</li>
<li>How much the features contribute to the performance of the model on unseen data a test data</li>
</ul>
</li>
</ul>
<h3 id="advantages-and-disadvantages-4">Advantages and disadvantages</h3>
<p>Advantages:</p>
<ul>
<li>Easy to understand</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li><strong>Because of the permutation</strong>, it can be <strong>computationally expensive</strong> to get the most accurate feature importances.</li>
<li>The <strong>permutation breaks the interaction not only with the outcome but also</strong> breaks the interaction <strong>with the other features</strong> (if any). Thus, <strong>the error increases not only due to the permutation</strong> of the feature <strong>but also due to the broken interaction</strong> with other features, reason why the <strong>results might be not related exclusively to the feature</strong> under examination. Then, the importance would not be exclusively of the feature under examination.</li>
<li>If features are correlated, the feature importance because the permutation would create <strong>unrealistic data instances</strong> (e.g  (2 meter person weighing 30 kg for example).</li>
<li>The permutation feature importance is <strong>computed based on the error</strong> of the model, but <strong>sometimes we might need to explain the importance of a feature based in other  criteria</strong>  than  the performance of the model. For example we might be interested in: how much of the prediction variate when the feature value changes, like partial dependent plots do .</li>
<li>You need access to the true outcome. If someone only provides you with the model and unlabeled data – but not the true outcome – you cannot compute the permutation feature importance.</li>
<li>The permutation feature importance <strong>depends on shuffling the feature</strong> , which adds randomness to the measurement. When the permutation is repeated, the <strong>results might vary greatly.</strong> <strong>However, repeating the permutation</strong> and averaging the importance measures over repetitions <strong>stabilizes the measure, but increases the time of computation.</strong></li>
</ul>
<h2 id="local-interpretable-model-agnostic-explanations-lime">Local interpretable model-agnostic explanations (LIME)</h2>
<h3 id="general-idea-4">General idea</h3>
<ul>
<li>LIME provides an explanation <strong>for a single data point.</strong></li>
<li>It tries to answer: which variables caused the prediction?</li>
<li>The technique attempts to <strong>understand the model by perturbing the input</strong> of data samples and understanding how the predictions change.</li>
<li>An <strong>explanation is created</strong> by approximating the underlying <strong>model locally by an interpretable one</strong>. Interpretable models are <strong>e.g. linear models</strong>, decision tree’s, etc. <strong>The interpretable models are trained on small perturbations</strong> of the original instance and should only provide a good local approximation.</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/SWNEjIyAQ_kI3Ie2uECO5eKDcqIG07lk7V-9sCOQyEpWGkVJt3GmO2xsqFEwCMXO9QgjRyM0vgnc=s500" alt="(Molnar 2018)"> (Molnar 2018)</p>
<p><strong>How does it work in general?</strong></p>
<ol>
<li>Permutation: from the labels of the training data take a sample (in python a normal distribution is assumed), see picture B.</li>
</ol>
<p><img src="https://lh3.googleusercontent.com/ay1hYKdkBb27Ny9ryoX6cSCyDNul0MUor2BndSJnHdVakT4mP0D79ZqCs4ML_84I1xtbmGnJeXm5=s900" alt="">(Molnar 2018)</p>
<ol start="2">
<li>Assign higher weight to points near the instance of interest</li>
</ol>
<p><img src="https://lh3.googleusercontent.com/CcDkl-J5ULSf6Y4BZ4KVUIlSu_ZkC4KlKUY2JUPzjIcbi3poWdRFn9EJyTL1sWbTOSY_cmUhgyn1=s900" alt="">(Molnar 2018)</p>
<ol start="3">
<li>Fit an interpretable model to the permuted data with the m features (from the permuted data weighted by its similarity to the original observation).</li>
</ol>
<p><img src="https://lh3.googleusercontent.com/CFXSdKkR1WGX_7RYfoRyw3FQG6JmtqhGUBBvmO6qbMUZaT5y1MLY8pSX1TowV4QXkc1IlSVcLtBh=s900" alt="enter image description here">(Molnar 2018)</p>
<ol start="4">
<li>Extract the feature weights from the simple model and use these as explanations for the complex models local behavior.</li>
</ol>
<h3 id="demo-5">Demo</h3>
<p>Generate the explanation for the first observation in the test dataset</p>
<pre class=" language-r"><code class="prism  language-r">i <span class="token operator">=</span> <span class="token number">6</span> <span class="token comment"># explain for observation 6th </span>

predictor_lime <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>xgboost_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">,</span> class <span class="token operator">=</span> <span class="token string">"likely"</span><span class="token punctuation">)</span> 
lime <span class="token operator">&lt;-</span> LocalModel<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_lime<span class="token punctuation">,</span> x.interest <span class="token operator">=</span> X<span class="token punctuation">[</span>i<span class="token punctuation">,</span> <span class="token punctuation">]</span><span class="token punctuation">,</span> k <span class="token operator">=</span> <span class="token number">7</span><span class="token punctuation">)</span>
plot<span class="token punctuation">(</span>lime<span class="token punctuation">)</span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/j7PYP1gxkmACGVl7SU2NNSIYDvO_fWAXRrHS4Z5bny3Ovmph1IBhgMYdsVkrixkLhL5LKaD0PsRK=s900" alt=""></p>
<p>Interpretation:</p>
<ul>
<li>The variable that caused the prediction is the “Recommedion letter strength”</li>
<li>While the varibles that  are</li>
</ul>
<p>This point  has been predicted as “likely” (class 2) to be accepted for the master program because:</p>
<ul>
<li>The  GPA  is  lower  than  8.13</li>
<li>The GRE score is between 309 and 317</li>
<li>The University rating  is  lower  than 2</li>
<li>…</li>
</ul>
<pre class=" language-r"><code class="prism  language-r">lime<span class="token operator">$</span>results
</code></pre>
<pre class=" language-r"><code class="prism  language-r"><span class="token comment">##                              beta x.recoded      effect x.original</span>
<span class="token comment">## GREScore             -0.0002521975    314.00 -0.07919001        314</span>
<span class="token comment">## TOEFLScore           -0.0079450623    103.00 -0.81834141        103</span>
<span class="token comment">## UniversityRating     -0.0904920219      2.00 -0.18098404          2</span>
<span class="token comment">## StatementOfPurpose   -0.0283988243      2.00 -0.05679765          2</span>
<span class="token comment">## RecommendationLetter  0.0521051108      3.00  0.15631533          3</span>
<span class="token comment">## CGPA                 -0.0777916701      8.21 -0.63866961       8.21</span>
<span class="token comment">## Research             -0.1428017233      0.00  0.00000000          0</span>
</code></pre>
<blockquote>
<p>lime$results</p>
</blockquote>
<h3 id="lime-in-detail"><strong>LIME in detail</strong></h3>
<ul>
<li>LIME provides local model interpretability. LIME modifies a single data sample by tweaking the feature values and observes the resulting impact on the output.</li>
<li>The output of LIME is a list of explanations, reflecting the contribution of each feature to the prediction of a data sample.</li>
<li>The ‘dataset’ is created by for example adding noise to continuous features, removing words or hiding parts of the image. By only approximating the black-box locally (in the neighborhood of the data sample) the task is significantly simplified.</li>
<li>LIME assumes that every complex model is linear on a local scale. We would expect two observations that are very similar (very close from each other) to output the same prediction regardless of how complex is the model.</li>
<li>LIME then states that it is possible to fit a simple model around a single observation that will mimic how the global model behaves at the space around my data point. The simple model can then be used to explain the predictions of the more complex model locally.</li>
</ul>
<p><strong>Permutation</strong><br>
LIME depends on the type of input data. Currently two types of inputs are supported: tabular and text</p>
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
<p><strong>Computing the similarity score</strong><br>
It is calculated differently according to whether it is for tabular or textual data</p>
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
<h3 id="advantages-and-disadvantages-5">Advantages and disadvantages</h3>
<p>Advantages:</p>
<ul>
<li>Provide a human-friendly  explanation and the idea of the algorithm is very intuitive, basically it applies a surrogate (linear) model in a local area of interest.</li>
<li>Works for  tabular  data, text, and  images</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li>LIME assumes that linear models can approximate local behaviour, which is an assumption that can be correct when it is about examining a very small region around the data sample. However, for a larger region the linear model might not be powerful enough to explain the behavior.</li>
<li>Also, if the  model is highly non-linear even for a very small region, then the exaplanation might not be a faithful.</li>
<li>It is not clear to which other instances  (or reagion) the explanation is valid.</li>
<li>LIME uses discretization for continuous predictors (regression cases), but discretization comes with an information loss.</li>
</ul>
<h2 id="anchors">Anchors</h2>
<h3 id="conceptualization-1">Conceptualization</h3>
<ul>
<li>An Anchor explains individual predictions with if-then rules. Such rules are intuitive to humans, and usually require low effort to comprehend and apply.</li>
<li>For example, the anchor in the following figure states that the model will almost always predict a Salary ≤ 50K if a person is not educated beyond high school, even if the other feature values would change.</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/M4Ype1yzyvv61H2GW1530lOmvU0tt2ONjhe-tR_meqLyC-5jtyiPJ854UzedFGeq3fRwzyOsqLt0=s600" alt="enter image description here"></p>
<ul>
<li>In short, an anchor is a rule that sufficiently “anchors” a prediction – such that changes to the rest of the instance do not matter  with a high probability.</li>
</ul>
<p>Anchors aims to improve pitfalls of LIME. It is developed by the same authors of LIME</p>
<ul>
<li>The linear LIME explanation brings some light into the reasons for the prediction, but it is not clear whether the same explanation can be applied to other instances. The question is then: what the local region is? With the LIME approach this is not easy to discover.</li>
<li>Anchors, on the other hand, make their coverage very clear, the user knows exactly when the explanation for an instance “generalizes” to other instances.</li>
<li>The assumption is that while the model is globally too complex to be explained succinctly, “zooming in” on individual predictions makes the explanation task feasible.<br>
<img src="https://lh3.googleusercontent.com/UICNP49fWJpyJx2QaV6D9MvCsYBgjKKi5Uu7nckdOhaqGL9RPTKPdELV7gRORCEvyOyxejEWckU0=s400" alt="enter image description here"></li>
</ul>
<p><strong>Toy example: predicting positive and negative sentiment</strong></p>
<p><img src="https://lh3.googleusercontent.com/MuRxjtNtRg-_6yl6wNVuIP0vjY5MEnA5h_YoTZWeLXh0xiKZrCcNXTQ0RjJzZjkLkocyr7UHBAo0=s600" alt="enter image description here"></p>
<ul>
<li>While LIME explanations provide insight into the model, their coverage is not clear, e.g. when does the word not have a positive influence on sentiment?</li>
<li>Anchors only apply when all the conditions in the rule are met (Figure part c). The anchors in Figure part c state that the presence of the words “not bad” ensure a prediction of positive sentiment</li>
<li>A piece of text that has the words not and bad will be likely predicted as positive even if the rest of the feature values change.</li>
<li>Anchors enable users to predict how a model would behave on unseen instances with higher precision.</li>
</ul>
<p><strong>Definition of an anchor:</strong></p>
<ul>
<li>Let f be the black box model</li>
<li>Let x be an instance (the one that we want to explain) that is perturbed by some perturbation distribution D.</li>
<li>Let A be the rule (set of predicates) that acts on x, such that A(x) returns 1 if all the predicates are true. For example, in past Figure: x = “This movie is not bad.”, f (x) = Positive, A = {“not”, “bad”}, then A(x) = 1.</li>
<li>Let D(·|A) denote the conditional distribution (perturbed distribution) where the rule A applies e.g. (e.g. similar texts where not and bad are present). E.g.:</li>
</ul>
<p><img src="https://lh3.googleusercontent.com/QuAikPS1uN-9HbLUZg14foyogNBkOxuaOxdU7P_8NAp8WZVhI_sPRshAHGqTuK_mTkqVJEb3m_7P=s300" alt="enter image description here"></p>
<ul>
<li>A is an anchor if A(x) = 1 (if all predicates of the rule are true) and if a sample z from D(z|A) (a z sample that follows the perturbed distribution and complies with rule A) has a high probability to be predicted as Positive. This would mean that f(x) = f(z), i.e.  the prediction of x is equal to the predictions on z. This  is formally stated as precision, where
<ul>
<li>precision(A) ≥ τ (predictions of z should be at least τ. τ is set by the user).</li>
</ul>
</li>
</ul>
<p>In short, an anchor A is a set of feature predicates on x that achieves a “high” probability of being predicted as positive.</p>
<p><strong>Perturbation:</strong></p>
<ul>
<li>At its core, the algorithm deploys a perturbation-based strategy. That means that the observed or explained instance gets perturbed, i.e., its feature values change according to some application-specific policy where the resulting data instances resemble neighbors of the initial instance. The policy in this case  is that the generated perturbations must comply with the rule A.</li>
<li>The resulting data instances are then evaluated using the model. However, querying often the model can be expensive. Also, there can be no exhaustive search in the case of continuous or sufficiently complex models. Reinforcement learning and its multi-armed bandits (MABs) provide a solution to this problem. They help to significantly reduce the number of samples required by using stochastic exploration approaches.</li>
</ul>
<p><strong>Anchors in deep</strong></p>
<p>There are two approaches to find anchors: bottom-up approach and beam search.</p>
<p>Bottom-up construction of Anchors:</p>
<ol>
<li>Initialize with an empty rule</li>
<li>Generate a number of candidate rules extending A by 1 additional feature predicate ai<br>
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>A</mi><mo>=</mo><mo>{</mo><mi>A</mi>&amp;ThinSpace;<mi mathvariant="normal">Λ</mi>&amp;ThinSpace;<msub><mi>a</mi><mi>i</mi></msub><mo separator="true">,</mo><mi>A</mi>&amp;ThinSpace;<mi mathvariant="normal">Λ</mi>&amp;ThinSpace;<msub><mi>a</mi><mrow><mi>i</mi><mo>+</mo><mn>1</mn></mrow></msub><mo separator="true">,</mo><mi>A</mi>&amp;ThinSpace;<mi mathvariant="normal">Λ</mi>&amp;ThinSpace;<msub><mi>a</mi><mrow><mi>i</mi><mo>+</mo><mn>2</mn></mrow></msub><mo separator="true">,</mo><mi mathvariant="normal">.</mi><mi mathvariant="normal">.</mi><mi mathvariant="normal">.</mi><mo>}</mo>&amp;ThinSpace;&amp;ThinSpace;<msub><mo>(</mo><mrow><mi>E</mi><mi>x</mi><mi>a</mi><mi>m</mi><mi>p</mi><mi>l</mi><mi>e</mi>&amp;ThinSpace;<mi>o</mi><mi>f</mi>&amp;ThinSpace;<mi>h</mi><mi>o</mi><mi>w</mi>&amp;ThinSpace;<mi>a</mi><mi>n</mi>&amp;ThinSpace;<mi>i</mi><mi>t</mi><mi>e</mi><mi>r</mi><mi>a</mi><mi>t</mi><mi>i</mi><mi>o</mi><mi>n</mi>&amp;ThinSpace;<mi>m</mi><mi>i</mi><mi>g</mi><mi>h</mi>&amp;ThinSpace;<mi>l</mi><mi>o</mi><mi>o</mi><mi>k</mi></mrow></msub><mo>)</mo></mrow><annotation encoding="application/x-tex"> A= \{A\, \Lambda\, a_i, A\, \Lambda\,  a_{i+1}, A\, \Lambda\,  a_{i+2}, ... \} \,\, (_{Example\, of\, how\, an\, iteration\, migh\, look})</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathit">A</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.03611em; vertical-align: -0.286108em;"></span><span class="mopen">{</span><span class="mord mathit">A</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord">Λ</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathit">a</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord mathit">A</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord">Λ</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathit">a</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">i</span><span class="mbin mtight">+</span><span class="mord mtight">1</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.208331em;"><span class=""></span></span></span></span></span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord mathit">A</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord">Λ</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathit">a</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight">i</span><span class="mbin mtight">+</span><span class="mord mtight">2</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.208331em;"><span class=""></span></span></span></span></span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord">.</span><span class="mord">.</span><span class="mord">.</span><span class="mclose">}</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mopen"><span class="mopen">(</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.336108em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span style="margin-right: 0.05764em;" class="mord mathit mtight">E</span><span class="mord mathit mtight">x</span><span class="mord mathit mtight">a</span><span class="mord mathit mtight">m</span><span class="mord mathit mtight">p</span><span style="margin-right: 0.01968em;" class="mord mathit mtight">l</span><span class="mord mathit mtight">e</span><span class="mspace mtight" style="margin-right: 0.195167em;"></span><span class="mord mathit mtight">o</span><span style="margin-right: 0.10764em;" class="mord mathit mtight">f</span><span class="mspace mtight" style="margin-right: 0.195167em;"></span><span class="mord mathit mtight">h</span><span class="mord mathit mtight">o</span><span style="margin-right: 0.02691em;" class="mord mathit mtight">w</span><span class="mspace mtight" style="margin-right: 0.195167em;"></span><span class="mord mathit mtight">a</span><span class="mord mathit mtight">n</span><span class="mspace mtight" style="margin-right: 0.195167em;"></span><span class="mord mathit mtight">i</span><span class="mord mathit mtight">t</span><span class="mord mathit mtight">e</span><span style="margin-right: 0.02778em;" class="mord mathit mtight">r</span><span class="mord mathit mtight">a</span><span class="mord mathit mtight">t</span><span class="mord mathit mtight">i</span><span class="mord mathit mtight">o</span><span class="mord mathit mtight">n</span><span class="mspace mtight" style="margin-right: 0.195167em;"></span><span class="mord mathit mtight">m</span><span class="mord mathit mtight">i</span><span style="margin-right: 0.03588em;" class="mord mathit mtight">g</span><span class="mord mathit mtight">h</span><span class="mspace mtight" style="margin-right: 0.195167em;"></span><span style="margin-right: 0.01968em;" class="mord mathit mtight">l</span><span class="mord mathit mtight">o</span><span class="mord mathit mtight">o</span><span style="margin-right: 0.03148em;" class="mord mathit mtight">k</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mclose">)</span></span></span></span></span></span></li>
<li>Estimate the precision of every candidate and identify the one with the highest precision, which will be used to extend A.</li>
<li>Iterate from step 2, or If the current candidate rule satisfy the precision constraint (precision(A) ≥ τ) with high probability, we have identified our desired anchor and terminate. It means that the following needs to be satisfied: <span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>P</mi><mo>(</mo><mi>p</mi><mi>r</mi><mi>e</mi><mi>c</mi><mo>(</mo><mi>A</mi><mo>)</mo><mo>≥</mo><mi>τ</mi><mo>)</mo><mo>≥</mo><mn>1</mn><mo>−</mo><mi>δ</mi><mo>)</mo></mrow><annotation encoding="application/x-tex"> P(prec(A) \ge \tau ) \ge 1 - \delta)  </annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span style="margin-right: 0.13889em;" class="mord mathit">P</span><span class="mopen">(</span><span class="mord mathit">p</span><span style="margin-right: 0.02778em;" class="mord mathit">r</span><span class="mord mathit">e</span><span class="mord mathit">c</span><span class="mopen">(</span><span class="mord mathit">A</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span style="margin-right: 0.1132em;" class="mord mathit">τ</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.72777em; vertical-align: -0.08333em;"></span><span class="mord">1</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span style="margin-right: 0.03785em;" class="mord mathit">δ</span><span class="mclose">)</span></span></span></span></span></span>
<ul>
<li>e.g. 90% probability that the evaluated/predicted points in sample z have a positive prediction 95% accurate.</li>
</ul>
</li>
</ol>
<p><img src="https://lh3.googleusercontent.com/Z_5mEkInapuwj5fRLR-rPvb0LuNHVgpJ9zu8HFR-DvbK3ld4AeiFRRlNTesVbTuoO52EL3K2_rOH=s900" alt="enter image description here"></p>
<p>Estimating precision:</p>
<ul>
<li>Anchors rely on samples from D(·|A) to efficiently estimate the precision of A, one sample per candidate. To estimate the fewest  samples (fewest  candidates), a multiarmed bandit algorithm is used:
<ul>
<li>Each candidate is an arm.</li>
<li>True precision of A on D(·|A) is  the latent reward (the  ones  that  come  clo)</li>
<li>Each pull of  the arm is an evaluation  of f(x) = f(z) on a sample from  D(z|A)<br>
The KL-LUCB (Kaufmann and Kalyanakrishnan) algorithm is used to identify the rule with the highest precision (refer to Ribeiro et al. 2018 for details about how this algorithm is used.)</li>
</ul>
</li>
</ul>
<p>Beam search:</p>
<ul>
<li>The Bottom-up approach has 2 major concerns: (1) due to the greedy nature of the approach, it is only able to maintain a single rule at a time and thus any suboptimal choice is irreversible, and (2) the algorithm does not try to achieve the highest coverage, instead returns the shortest anchor (regarding the number of predicates) that it finds. Nevertheless, note that the findings show that short anchors are likely to have a higher coverage.<br>
•In order to address both these concerns, a greedy approach is extended to perform a beam-search (a strategy to choose better results from all possible candidates) by maintaining a set of candidate rules, while guiding the search to identify amongst many possible anchors the one that has the highest coverage (refer to Ribeiro et al. 2018 for details about how this algorithm is used) .<br>
•It is expected that this approach will likely identify an anchor with higher coverage than bottom-up search.</li>
</ul>
<h3 id="advantages-and-disadvantages-6">Advantages and disadvantages</h3>
<p>Advantages:</p>
<ul>
<li>An important advantage of anchors is that it expresses the explanation in short, disjoint rules, which are easier to interpret</li>
<li>The user knows when the explanation for an instance generalizes to other instances.</li>
</ul>
<p>Disadvantages:</p>
<ul>
<li>Only captures the behavior of the model on a local region.</li>
<li>Anchors can create explanations for complex functions, but the rule set to explain the prediction can become large.</li>
</ul>
<h2 id="shapley-values">Shapley values</h2>
<h3 id="conceptualization-2">Conceptualization</h3>
<ul>
<li>Shapley values is a local explanation approach, this is, it explain the prediction of a specific instance.</li>
<li>It is a game theory-based approach where feature values of an instance are players in a competitive game. Each player looks for receiving a payoff, which is the prediction. A player can act alone or in coalition with other players, in this case, the group of players receive one payoff.  Shapley values explains how much the feature value i contribute to the prediction compared to the average predictions of all data.
<ul>
<li>Game: the prediction task for a single instance.</li>
<li>Players: the feature values of the instance that collaborated to receive the gain (to make the prediction).</li>
<li>Coalition: combination of features</li>
<li>Payoff: prediction that a coalition receives</li>
<li>Gain: actual prediction minus average prediction for all instances.</li>
</ul>
</li>
<li>Formally, a Shapley value (ϕ) is the average marginal contribution of a feature value to the prediction over all possible coalitions. A Shapley value for a certain feature value  i , in a model with n total features, given a prediction p is: <span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>ϕ</mi><mo>(</mo><mi>p</mi><mo>)</mo><mo>=</mo><munder><mo>∑</mo><mrow><mi>S</mi><mo>⊆</mo><mi>N</mi><mi mathvariant="normal">/</mi><mi>i</mi></mrow></munder><mfrac><mrow><mi mathvariant="normal">∣</mi><mi>S</mi><mi mathvariant="normal">∣</mi><mo>!</mo><mo>(</mo><mi>n</mi><mo>−</mo><mi mathvariant="normal">∣</mi><mi>S</mi><mi mathvariant="normal">∣</mi><mo>−</mo><mn>1</mn><mo>)</mo><mo>!</mo></mrow><mrow><mi>n</mi><mo>!</mo></mrow></mfrac><mo>(</mo><mi>p</mi><mo>(</mo><mi>s</mi><mo>∪</mo><mi>i</mi><mo>)</mo><mo>−</mo><mi>p</mi><mo>(</mo><mi>S</mi><mo>)</mo><mo>)</mo></mrow><annotation encoding="application/x-tex">  \phi (p)=\sum_{S\subseteq N/i} \frac{ |S|!(n - |S| -1)!}{n!}(p(s \cup i) - p(S))</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit">ϕ</span><span class="mopen">(</span><span class="mord mathit">p</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 2.94301em; vertical-align: -1.51601em;"></span><span class="mop op-limits"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.05001em;"><span class="" style="top: -1.809em; margin-left: 0em;"><span class="pstrut" style="height: 3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span style="margin-right: 0.05764em;" class="mord mathit mtight">S</span><span class="mrel mtight">⊆</span><span style="margin-right: 0.10903em;" class="mord mathit mtight">N</span><span class="mord mtight">/</span><span class="mord mathit mtight">i</span></span></span></span><span class="" style="top: -3.05001em;"><span class="pstrut" style="height: 3.05em;"></span><span class=""><span class="mop op-symbol large-op">∑</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 1.51601em;"><span class=""></span></span></span></span></span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.427em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord mathit">n</span><span class="mclose">!</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.677em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">∣</span><span style="margin-right: 0.05764em;" class="mord mathit">S</span><span class="mord">∣</span><span class="mclose">!</span><span class="mopen">(</span><span class="mord mathit">n</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mord">∣</span><span style="margin-right: 0.05764em;" class="mord mathit">S</span><span class="mord">∣</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mord">1</span><span class="mclose">)</span><span class="mclose">!</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mopen">(</span><span class="mord mathit">p</span><span class="mopen">(</span><span class="mord mathit">s</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">∪</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit">i</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit">p</span><span class="mopen">(</span><span style="margin-right: 0.05764em;" class="mord mathit">S</span><span class="mclose">)</span><span class="mclose">)</span></span></span></span></span></span></li>
<li>In a very simple way, this equation computes what the prediction of the model would be without the feature value i, .Then computes the prediction of the model with feature value i (feature effect), and finally calculates the difference, which corresponds to the contribution:
<ul>
<li>Importance of i = p(with i) – p(without i)</li>
</ul>
</li>
</ul>
<p><strong>Spliting the payoff between the players</strong></p>
<p>One important question to answer is about how to split fairly the payoff when players with different skills act in a coalition. Some conditions are first set to make sure that the payoff is being divided fairly:</p>
<ol>
<li>The sum of what every player receives should equal to the total reward</li>
<li>If two players contributed the same value, then they should receive the same amount from the reward</li>
<li>Someone who contributed no value should receive nothing</li>
<li>If the coalition plays two games, then an player’s reward from both games should equal its reward from their first game plus its reward from the second game</li>
</ol>
<p>But again, <strong>how to split the payoff fairly?</strong> A possible solutions is according to the sequence of how group members joined, and tracking marginal contributions of each player.<br>
This would mean that every player would be rewarded for what they contribute to achieve the total payoff, and according to the the order in which a player joined. For example:</p>
<blockquote>
<p>“If Ava was the first member of the group, with a payoff of 5, and Bill joined to bring the payoff to 9, and later Christine joined to bring the payoff to 11, then the players’ respective payoffs would be (5, 4, 2). But, what if Christine and Bill have very similar skill sets? Then, it might the case that Christine would have a higher marginal contribution if she joined the group before Bill, because she’d be the first one to provide their overlapping skill set, and then when he joined, his marginal contribution would be lower. imagine a different payoff vector for (Ava, Bill, Christine) of (5, 1, 5)"<br>
<a href="https://towardsdatascience.com/one-feature-attribution-method-to-supposedly-rule-them-all-shapley-values-f3e04534983d">https://towardsdatascience.com/one-feature-attribution-method-to-supposedly-rule-them-all-shapley-values-f3e04534983d</a></p>
</blockquote>
<p>Let´s see this problem in the following toy example: given the age and gender of an individual, we predict whether or not they will like computer games.</p>
<p><img src="https://lh3.googleusercontent.com/bR5YreBzvcRqwQz_rveIvOriLMO1xZVbyGThgm14g2Gt4TWYSat6QatswvEeAXBmuB2NdQxdBnFE=s600" alt=""></p>
<ol>
<li>
<p>Sequence (age, gender) for Bobby-14:</p>
<ul>
<li>When the model sees Bobby’s age at first, it will take him left on the first split.</li>
<li>As gender is still not identified, the average of the leaves below will be assigned: (2 + 0.1) / 2 = 1.05 (effect of the age)</li>
<li>Then, when the model sees he is a male, the model assigns him a score of 2: 2 – 1.05 = 0.95 (effect of the gender)<br>
Therefore, ϕAgeBobby=1.05 and ϕGenderBobby=0.95.</li>
</ul>
</li>
<li>
<p>Sequence (gender, age) for Bobby-14:</p>
</li>
</ol>
<ul>
<li>At first the model does not have an age to split on, it then has to take the average of all leaves:
<ul>
<li>The average of leaves from is male? is: (2 + 0.1) / 2 = 1.05</li>
<li>The total average with the remaining leaf is: (1.05 + (-1)) / 2 = 0.025 (effect of the gender)</li>
</ul>
</li>
<li>Then, when the model sees he is 14, it assigns him a score of 2, the effect of age is then: (2–0.025)=1.975 (effect of the age)</li>
<li>Therefore, for this sequence: ϕAgeBobby=1.975 and ϕGenderBobby=0.025, which is different from the past sequence</li>
</ul>
<p><strong>Which of these is a fair payoff for the feature value of AgeBobby, 1.05 or 1.975?, and for GenderBobby?.</strong></p>
<p>Shapley values solve the problem by finding each player’s marginal contribution, averaged over every possible sequence in which the players could have been added to the group. For instance, for the previus example about Ava, Bill, and Christine´s payoff, all the possible sequences are: ABC, ACB, BCA, BAC, CAB, and CBA. For each player we can compute its marginal payoff in every sequence and the average them to make it “fair”.</p>
<p>Shapley value considers all possible values by calculating a weighted sum to find a final value. It means that it does not really compute for all sequences (e.g (age, gender) and (gender, age)) but computes a weighted sum.</p>
<p><strong>But how are the the weights assigned to each component of the sum?</strong></p>
<ul>
<li>It considers how many different permutations in the sequence vector exist by taking into account the features which are in the set S (this is done by the ∣S∣! ) as well as the features that still have to be added ( by the (n−∣S∣−1)!). What is important to know here is</li>
<li>Finally, everything is normalized by the features we have in total in the coalition (n).<br>
<img src="https://lh3.googleusercontent.com/bTg-d8VCUoMd7SFroXpLMQlueCYqgBAVH07B0i1er-rH-gnbw03NXhLPbG0OlkgXlAbXZXB4pLIv=s600" alt="enter image description here"><br>
<strong>Computing a Shapley Value</strong></li>
</ul>
<ol>
<li>Build sets S (all possible feature combinations of Bobby, excluding his age). As he has only one feature left (gender) then all the possible coalitions are: {gender}, and an empty set {}. If we would have more features, as for instance age, gender, and occupation, then all the possible coalitions would be: {age}, {gender}, {occupation}, {age, gender}, {age, occupation}, {gender, occupation}, {age, gender, occupation}, and an empty set {}.</li>
<li>Calculate the Shapley value ϕ_i § for each sets S.
<ol>
<li>Shapley value for S = {}:
<ul>
<li>∣S∣ = 0, ∣S∣! = 1</li>
<li>n = 2 (features)</li>
<li>p(S) = the model’s prediction when it sees no features is the average of the leaves (average effect): 0.025</li>
<li>p(S ∪ i ) = the prediction of the model when it sees only the age (we calculated this before): 1.05</li>
<li>Shapley value is then: 0.5125</li>
</ul>
</li>
<li>Shapley value for S = {gender}:
<ul>
<li>∣S∣ = 1, ∣S∣! = 1</li>
<li>n =</li>
<li>p(S) = the model’s prediction when it sees only gender is the average effect (we calculated this before): 0.025</li>
<li>p(S ∪ i ) = the prediction of the model when it sees the gender and then when it sees the age (we calculated this before): 2</li>
<li>Shapley value is then: 0.9875</li>
</ul>
</li>
</ol>
</li>
<li>Add the values together: ϕBobby-14 = 0.5125 + 0.9875 = 1.5</li>
</ol>
<ul>
<li>Remember that the contribution is the difference between the feature effect minus the average effect. It means that Bobby´s age-14 makes the prediction to be 1.5 over the average prediction of all people. Imagine that the average prediction of all people is 0.25 and 2 represents the class: likes video games. Then his age makes the probability that he likes video games to be higher than the average person by 1.5.</li>
</ul>
<p>In <strong>summary,</strong> Shapley values calculate the importance of a feature by comparing what a model predicts with and without the feature. However, since the order in which a model sees features can affect its predictions, this is done in every possible order, so that the features are fairly compared.</p>
<h3 id="demo-6">Demo</h3>
<pre class=" language-r"><code class="prism  language-r">predictor_shapley <span class="token operator">&lt;-</span> Predictor<span class="token operator">$</span>new<span class="token punctuation">(</span>rf_model<span class="token punctuation">,</span> data <span class="token operator">=</span> X<span class="token punctuation">,</span> type <span class="token operator">=</span> <span class="token string">"prob"</span><span class="token punctuation">,</span>class <span class="token operator">=</span> <span class="token number">2</span><span class="token punctuation">)</span>
shapley_rf <span class="token operator">&lt;-</span> Shapley<span class="token operator">$</span>new<span class="token punctuation">(</span>predictor_shapley<span class="token punctuation">,</span> x.interest <span class="token operator">=</span> X<span class="token punctuation">[</span><span class="token number">5</span><span class="token punctuation">,</span> <span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token comment"># explain the fifth obeservation of the dataset</span>
plot<span class="token punctuation">(</span>shapley_rf<span class="token punctuation">)</span> <span class="token comment">#plot </span>
</code></pre>
<p><img src="https://lh3.googleusercontent.com/QhgzxlIjeJH0GhYAwQL9t-4BqBiZavdnpoYyTXZtuVrbwkfPEH8FQz9q_rIIt-dmLHPQu9lmpmg8=s900" alt="enter image description here"><br>
From the results we can interpret:</p>
<ul>
<li>The difference between the actual prediction and the average prediction is 0.36. Note that the sum of the contributions yields 0.36</li>
<li>This person has 0.36 more chance to be predicted as “likely” (class 2) to be accepted for the master program compared to the average people in class 2.</li>
<li>The CPGA score of 8.21 increased the chance the most (It increased  the  probability of being classified as “likely” in ~ 0.09 above  the  average  aspirants (0.63)).</li>
</ul>
<h3 id="advantages-and-disadvantages-7">Advantages and disadvantages</h3>
<p>Advantages:</p>
<ul>
<li>The difference between the prediction and the average prediction is fairly distributed among the feature values of the instance</li>
<li>Compared with many of the other approaches, shapley values take into consideration the relationships between the features</li>
</ul>
<p>Disadvantages:</p>
<h1 id="use-cases">Use cases</h1>
<ul>
<li>Instant explanation: implementation together with recommender systems. Individual more willing to buy something if it is explained why a product is being recommended. E.g. 2 a bank that explains why a customer can’t get a loan, and what can s/he do to get one (credit rating).</li>
<li>Historical Explanation: consists of storing explanations as a proof that supports the reasons of a decision  that was taken.</li>
<li>Qualitative assessment: to evaluate the quality of the criteria taken by the algorithm for the classification.</li>
<li>Detect biased or manipulated predictions (identify problems with the models)</li>
<li>Legal requirements: to fulfil external legal requirements or internal compliances. In the future —&gt; the right to explanation of automated decision-making</li>
<li>Ethical assessment: discrimination caused by AI, see the following example from the Amazon company:
<ul>
<li>The retailer Amazon experimented with an AI recruiting tool that showed bias against women. This tool was developed to assist the human resource department for classifying  job applicants according to their suitability.</li>
<li>It was revealed that the model penalized applicants with a resume containing words like female or woman. Because of that, Amazon adjusted the model to be neutral to gender-related terminology but decided nevertheless to shut down the project since there was no guarantee that the model did not find other ways to discriminate applicants.</li>
</ul>
</li>
<li>Security assessment: detect manipulated data that attempt to change the results of the predictions.</li>
</ul>
<h1 id="references">References</h1>
<p>Molnar, C. (2018). Interpretable machine learning: A guide for making black box models explainable. <em>Christoph Molnar, Leanpub</em>.</p>

