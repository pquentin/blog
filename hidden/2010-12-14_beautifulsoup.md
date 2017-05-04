Title: BeautifulSoup
Date: 2010-12-14T00:20:19+01+00
Category: Logiciel
status: hidden

<p>BeautifulSoup est en fait un parseur html. Sa particularité est d'utiliser les mêmes heuristiques que nos navigateurs web pour réussir à obtenir quelque chose de documents pourris où les tags sont pas fermés par exemple. Ça permet donc d'avoir un parser robuste et acceptable pour parser n'importe quelle page web. Comme d'habitude un petit exemple de code ; celui-ci affiche la table des matières de "Erlang Programming" en allant la chercher sur le site d'O'Reilly :</p>

<div class="highlight"><pre><span class="c">#!/usr/bin/python</span>
<span class="c"># -*- coding: utf-8 -*-</span>

<span class="kn">from</span> <span class="nn">BeautifulSoup</span> <span class="kn">import</span> <span class="n">BeautifulSoup</span><span class="p">,</span> <span class="n">BeautifulStoneSoup</span>
<span class="kn">import</span> <span class="nn">re</span><span class="o">,</span> <span class="nn">urllib2</span>

<span class="n">html</span> <span class="o">=</span> <span class="n">urllib2</span><span class="o">.</span><span class="n">urlopen</span><span class="p">(</span><span class="s">&#39;http://oreilly.com/catalog/9780596518189&#39;</span><span class="p">)</span><span class="o">.</span><span class="n">read</span><span class="p">()</span>
<span class="n">page</span> <span class="o">=</span> <span class="n">BeautifulSoup</span><span class="p">(</span><span class="n">html</span><span class="p">,</span> <span class="n">convertEntities</span><span class="o">=</span><span class="n">BeautifulStoneSoup</span><span class="o">.</span><span class="n">HTML_ENTITIES</span><span class="p">)</span>

<span class="n">titre</span> <span class="o">=</span> <span class="n">page</span><span class="o">.</span><span class="n">find</span><span class="p">(</span><span class="s">&#39;meta&#39;</span><span class="p">,</span> <span class="p">{</span><span class="s">&#39;name&#39;</span><span class="p">:</span> <span class="s">&#39;book_title&#39;</span><span class="p">})[</span><span class="s">&#39;content&#39;</span><span class="p">]</span>
<span class="k">print</span> <span class="n">titre</span>

<span class="k">for</span> <span class="n">chapitre</span> <span class="ow">in</span> <span class="n">page</span><span class="o">.</span><span class="n">findAll</span><span class="p">(</span><span class="s">&#39;li&#39;</span><span class="p">,</span> <span class="p">{</span><span class="s">&#39;class&#39;</span><span class="p">:</span> <span class="s">&#39;chapter&#39;</span><span class="p">}):</span>
    <span class="n">titre_raw</span> <span class="o">=</span> <span class="n">chapitre</span><span class="o">.</span><span class="n">find</span><span class="p">(</span><span class="s">&#39;h3&#39;</span><span class="p">)</span><span class="o">.</span><span class="n">contents</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span>
    <span class="n">titre</span> <span class="o">=</span> <span class="n">re</span><span class="o">.</span><span class="n">sub</span><span class="p">(</span><span class="s">r&#39;\s+&#39;</span><span class="p">,</span> <span class="s">&#39; &#39;</span><span class="p">,</span> <span class="n">titre_raw</span><span class="o">.</span><span class="n">strip</span><span class="p">())</span>
    <span class="k">print</span> <span class="s">&#39;- </span><span class="si">%s</span><span class="s">&#39;</span> <span class="o">%</span> <span class="n">titre</span>
    <span class="k">for</span> <span class="n">section</span> <span class="ow">in</span> <span class="n">chapitre</span><span class="o">.</span><span class="n">findAll</span><span class="p">(</span><span class="s">&#39;li&#39;</span><span class="p">,</span> <span class="p">{</span><span class="s">&#39;class&#39;</span><span class="p">:</span> <span class="s">&#39;sect1&#39;</span><span class="p">}):</span>
        <span class="k">print</span> <span class="s">&#39;  - </span><span class="si">%s</span><span class="s">&#39;</span> <span class="o">%</span> <span class="n">section</span><span class="o">.</span><span class="n">contents</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="o">.</span><span class="n">string</span>
</pre></div>

<p>Agréable non ? Si c'est pas lisible immédiatement, un petit coup d'oeil sur le code html de la page web, et vous verrez que si, c'est pas mal. Ça permet vraiment de donner le boulot chiant à BeautifulSoup. Pour plus de détails sur la librairie, il suffit d'avoir voir la <a href="http://www.crummy.com/software/BeautifulSoup/documentation.html">documentation</a>, formulée de manière un peu inhabituelle mais tout de même efficace.</p>

<p>En plus c'est <a href="http://heymans.tumblr.com/post/2080834363/just-found-this-gem-in-the-beautiful-soup-source">enterprise ready</a>. :)</p>
