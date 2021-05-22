---
toc: true
layout: post
description: FOMO and similarity search
categories: [similarity, reference]
title: Fingerprint similarity thresholds for database searches
image: images/blog/similarity-search-threshold1-img1.png
---

**Updated 22.05.2021 after I ran the background retrieval experiments using 5K
random molecules from the "Related" set instead of the 1K in the original post.
This doesn't have a major impact on results.

# Prologue
If you're interested in this topic and have questions, notice mistakes in my
analysis, or have suggestions or ideas for improving this (particularly when it
comes to the sets of "related compounds" I use), please either send me email or
leave a comment. 

# Intro / Results

One of the RDKit blog posts I refer back to the most is the one where I tried to
establish the Tanimoto similarity value which constitutes a "noise level" for
each of the fingerprints the RDKit supports by looking at the distributions of
similarities between randomly chosen molecules. I periodically update the post
just in case the threshold values change with RDKit versions. Here's [the most
recent version of the
post](https://greglandrum.github.io/rdkit-blog/fingerprints/similarity/reference/2021/05/18/fingerprint-thresholds1.html)
and the [associated jupyter
notebook](https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Fingerprint%20Thresholds.ipynb).
I find it really useful to be able to say things like "The 95% noise level for
Tanimoto similarity calculated with the the bit-based version of the RDKit's
MFP2 is 0.27." Based on this I know that when doing similarity searches the
threshold for MFP2 shouldn't be set below 0.27 (I normally say 0.3) in most cases.
But that analysis doesn't tell me what I should set the threshold to.

Of course the answer to that question is "it depends". Let's assume that the
database you're searching contains a certain number of compounds which would
actually be interesting for you and a much larger number of compounds which are
not interesting (at least not for the search you're currently running). And
let's further assume that the similarities between those interesting compounds
and your query is generally above the noise level for the fingerprint you're
using. Any similarity search is going to return a mix of both interesting and
non-interesting compounds and the proportions in that mix are generally going to
be determined by the similarity threshould you use. Setting the similarity
threshold high tends to give a larger proportion of interesting compounds at
the cost of missing interesting compounds while a lower threshold will return
more of the interesting compounds but a higher fraction of uninteresting
compounds. 

Basically how bad your FOMO is will determine how many results you need to look
through. 

This isn't a big deal if you're searching a small database and or if you're
going to be post-processing the results using some other computational tool, but
if the idea is that you're going to actually be *looking* at the results of the
similarity search, then result sets with 10K or more rows are going to require a
lot of patience.

This post is an attempt to come up with recommendations for reasonable threshold
values for the common RDKit fingerprints so that you can make a more informed
decision about what to use for a given search.

There's a more complete description below along with links to the jupyter
notebooks with the actual code, but here's a quick summary of what I did:
1. I started with 301 groups of related compounds (~20K compounds in all). Each
   group is a set of 50-100 compounds from a single ChEMBL document.
2. I calculated intra-group similarities within each of those 301 groups using
   each of the fingerprint types to determine thresholds for retrieving various
   fractions of each group.
3. I used that same set of 20K compounds to do similarity searches on 100K
   molecules randomly selected from ChEMBL in order to determine what fraction
   of the database would be retrieved for various similarity thresholds.

Here are what the results look like for bit-based MFP2:
<table>
<tr><th></th> <th></th> <th colspan="2">0.95 of related compounds</th> <th colspan="2">0.9 of related compounds</th> <th colspan="2">0.8 of related compounds</th> <th colspan="2">0.5 of related compounds</th></tr>
<tr><th>Fingerprint</th> <th>0.95 noise level</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th></tr>
<tr>
<td><b>Morgan2 (bits)</b></td> <td>0.27</td> <td>0.35</td> <td>0.0036 / 3567</td> <td>0.4</td> <td>0.00023 / 230</td> <td>0.45</td> <td>0.00014 / 135</td> <td>0.55</td> <td>2.5e-05 / 25</td> </tr>
</table>

The 0.95 noise level (from the previous analysis) for this FP is 0.27. If I want
to retrieve 95% of the related compounds I need to set the similarity threshold
to 0.35. With this threshold I would retrieve ~3500 compounds per million
compounds in the database (0.4% of the database). Similarly, if I were willing
to live with finding 50% of the related actives I could set the search threshold
to 0.55, in which case I'd only retrieve ~25 rows per million compounds in the
database.

I find this is a useful way of thinking about the thresholds: it makes the
balance between recall (number of interesting compounds retrieved) and the
overall result set size visible. For example, for the MFP2 results shown above,
if I'm willing to live with retrieving about 90% of the interesting compounds
instead of 95% I would only have to look through about 1/20th of the results
from the database.


With that explained, here's the full results table:
<table>
<tr><th></th> <th></th> <th colspan="2">0.95 of related compounds</th> <th colspan="2">0.9 of related compounds</th> <th colspan="2">0.8 of related compounds</th> <th colspan="2">0.5 of related compounds</th></tr>
<tr><th>Fingerprint</th> <th>0.95 noise level</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th></tr>
<tr>
<td><b>MACCS</b></td> <td>0.57</td> <td>0.6</td> <td>0.043 / 42650</td> <td>0.65</td> <td>0.023 / 22840</td> <td>0.7</td> <td>0.003 / 3030</td> <td>0.8</td> <td>0.0001 / 100</td> </tr>
<tr>
<td><b>Morgan0 (counts)</b></td> <td>0.57</td> <td>0.55</td> <td>0.058 / 58467</td> <td>0.6</td> <td>0.014 / 14240</td> <td>0.65</td> <td>0.0076 / 7570</td> <td>0.75</td> <td>0.00047 / 470</td> </tr>
<tr>
<td><b>Morgan1 (counts)</b></td> <td>0.36</td> <td>0.45</td> <td>0.0042 / 4230</td> <td>0.5</td> <td>0.00032 / 320</td> <td>0.55</td> <td>0.00018 / 180</td> <td>0.65</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>Morgan2 (counts)</b></td> <td>0.25</td> <td>0.35</td> <td>0.0018 / 1802</td> <td>0.4</td> <td>0.00017 / 170</td> <td>0.45</td> <td>0.00011 / 105</td> <td>0.55</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>Morgan3 (counts)</b></td> <td>0.20</td> <td>0.3</td> <td>0.00032 / 320</td> <td>0.3</td> <td>0.00032 / 320</td> <td>0.35</td> <td>0.00018 / 184</td> <td>0.45</td> <td>3.5e-05 / 35</td> </tr>
<tr>
<td><b>Morgan0 (bits)</b></td> <td>0.57</td> <td>0.55</td> <td>0.062 / 61650</td> <td>0.6</td> <td>0.016 / 15515</td> <td>0.65</td> <td>0.0083 / 8250</td> <td>0.75</td> <td>0.00051 / 512</td> </tr>
<tr>
<td><b>Morgan1 (bits)</b></td> <td>0.37</td> <td>0.45</td> <td>0.0052 / 5217</td> <td>0.5</td> <td>0.00038 / 380</td> <td>0.55</td> <td>0.00021 / 210</td> <td>0.65</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>Morgan2 (bits)</b></td> <td>0.27</td> <td>0.35</td> <td>0.0036 / 3567</td> <td>0.4</td> <td>0.00023 / 230</td> <td>0.45</td> <td>0.00014 / 135</td> <td>0.55</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>Morgan3 (bits)</b></td> <td>0.22</td> <td>0.3</td> <td>0.00063 / 630</td> <td>0.35</td> <td>0.00034 / 344</td> <td>0.35</td> <td>0.00034 / 344</td> <td>0.5</td> <td>2e-05 / 20</td> </tr>
<tr>
<td><b>FeatMorgan0 (counts)</b></td> <td>0.74</td> <td>0.6</td> <td>0.25 / 246595</td> <td>0.65</td> <td>0.16 / 157367</td> <td>0.7</td> <td>0.068 / 68140</td> <td>0.8</td> <td>0.0082 / 8195</td> </tr>
<tr>
<td><b>FeatMorgan1 (counts)</b></td> <td>0.51</td> <td>0.55</td> <td>0.021 / 21037</td> <td>0.55</td> <td>0.021 / 21037</td> <td>0.6</td> <td>0.0026 / 2610</td> <td>0.7</td> <td>0.00013 / 130</td> </tr>
<tr>
<td><b>FeatMorgan2 (counts)</b></td> <td>0.36</td> <td>0.45</td> <td>0.004 / 4050</td> <td>0.5</td> <td>0.00027 / 270</td> <td>0.55</td> <td>0.00016 / 155</td> <td>0.65</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>FeatMorgan3 (counts)</b></td> <td>0.28</td> <td>0.35</td> <td>0.0054 / 5412</td> <td>0.4</td> <td>0.00028 / 280</td> <td>0.45</td> <td>0.00016 / 165</td> <td>0.55</td> <td>3.5e-05 / 35</td> </tr>
<tr>
<td><b>FeatMorgan0 (bits)</b></td> <td>0.74</td> <td>0.6</td> <td>0.25 / 246595</td> <td>0.65</td> <td>0.16 / 157367</td> <td>0.7</td> <td>0.068 / 68140</td> <td>0.8</td> <td>0.0082 / 8195</td> </tr>
<tr>
<td><b>FeatMorgan1 (bits)</b></td> <td>0.51</td> <td>0.55</td> <td>0.023 / 23195</td> <td>0.55</td> <td>0.023 / 23195</td> <td>0.6</td> <td>0.003 / 3010</td> <td>0.7</td> <td>0.00014 / 140</td> </tr>
<tr>
<td><b>FeatMorgan2 (bits)</b></td> <td>0.38</td> <td>0.45</td> <td>0.0065 / 6452</td> <td>0.5</td> <td>0.00036 / 360</td> <td>0.55</td> <td>0.00021 / 205</td> <td>0.65</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>FeatMorgan3 (bits)</b></td> <td>0.30</td> <td>0.4</td> <td>0.00045 / 450</td> <td>0.4</td> <td>0.00045 / 450</td> <td>0.45</td> <td>0.00025 / 250</td> <td>0.55</td> <td>3.5e-05 / 35</td> </tr>
<tr>
<td><b>RDKit 4 (bits)</b></td> <td>0.33</td> <td>0.5</td> <td>0.0011 / 1120</td> <td>0.55</td> <td>0.00065 / 645</td> <td>0.6</td> <td>0.00017 / 170</td> <td>0.7</td> <td>4e-05 / 40</td> </tr>
<tr>
<td><b>RDKit 5 (bits)</b></td> <td>0.29</td> <td>0.5</td> <td>0.0004 / 400</td> <td>0.55</td> <td>0.00025 / 245</td> <td>0.6</td> <td>9e-05 / 90</td> <td>0.7</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>RDKit 6 (bits)</b></td> <td>0.31</td> <td>0.5</td> <td>0.00027 / 270</td> <td>0.55</td> <td>0.00017 / 170</td> <td>0.6</td> <td>7e-05 / 70</td> <td>0.7</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>RDKit 7 (bits)</b></td> <td>0.43</td> <td>0.5</td> <td>0.0011 / 1150</td> <td>0.6</td> <td>9e-05 / 90</td> <td>0.65</td> <td>6e-05 / 60</td> <td>0.75</td> <td>2e-05 / 20</td> </tr>
<tr>
<td><b>linear RDKit 4 (bits)</b></td> <td>0.35</td> <td>0.5</td> <td>0.0023 / 2255</td> <td>0.55</td> <td>0.0013 / 1277</td> <td>0.6</td> <td>0.0003 / 300</td> <td>0.75</td> <td>4e-05 / 40</td> </tr>
<tr>
<td><b>linear RDKit 5 (bits)</b></td> <td>0.31</td> <td>0.45</td> <td>0.0036 / 3590</td> <td>0.55</td> <td>0.0004 / 400</td> <td>0.6</td> <td>0.00013 / 130</td> <td>0.7</td> <td>4e-05 / 40</td> </tr>
<tr>
<td><b>linear RDKit 6 (bits)</b></td> <td>0.28</td> <td>0.45</td> <td>0.0015 / 1525</td> <td>0.5</td> <td>0.00033 / 330</td> <td>0.6</td> <td>8e-05 / 80</td> <td>0.7</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>linear RDKit 7 (bits)</b></td> <td>0.26</td> <td>0.45</td> <td>0.00076 / 755</td> <td>0.5</td> <td>0.0002 / 200</td> <td>0.55</td> <td>0.00013 / 130</td> <td>0.7</td> <td>2e-05 / 20</td> </tr>
<tr>
<td><b>Atom Pairs (counts)</b></td> <td>0.27</td> <td>0.3</td> <td>0.0057 / 5730</td> <td>0.35</td> <td>0.0029 / 2949</td> <td>0.4</td> <td>0.00017 / 170</td> <td>0.5</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>Topological Torsions (counts)</b></td> <td>0.19</td> <td>0.3</td> <td>0.00078 / 780</td> <td>0.35</td> <td>0.00045 / 449</td> <td>0.45</td> <td>8e-05 / 80</td> <td>0.55</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>Atom Pairs (bits)</b></td> <td>0.36</td> <td>0.4</td> <td>0.009 / 8960</td> <td>0.45</td> <td>0.0045 / 4545</td> <td>0.5</td> <td>0.00013 / 130</td> <td>0.55</td> <td>7.5e-05 / 75</td> </tr>
<tr>
<td><b>Topological Torsions (bits)</b></td> <td>0.22</td> <td>0.35</td> <td>0.00093 / 934</td> <td>0.4</td> <td>0.00017 / 170</td> <td>0.45</td> <td>0.00011 / 105</td> <td>0.55</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>Avalon 512 (bits)</b></td> <td>0.51</td> <td>0.6</td> <td>0.00082 / 820</td> <td>0.65</td> <td>0.00046 / 455</td> <td>0.7</td> <td>9e-05 / 90</td> <td>0.8</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>Avalon 1024 (bits)</b></td> <td>0.37</td> <td>0.55</td> <td>0.00083 / 830</td> <td>0.6</td> <td>0.00017 / 170</td> <td>0.65</td> <td>0.00011 / 105</td> <td>0.75</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>Avalon 512 (counts)</b></td> <td>0.42</td> <td>0.5</td> <td>0.0042 / 4180</td> <td>0.55</td> <td>0.0022 / 2215</td> <td>0.65</td> <td>0.00015 / 145</td> <td>0.75</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>Avalon 1024 (counts)</b></td> <td>0.38</td> <td>0.5</td> <td>0.0019 / 1890</td> <td>0.55</td> <td>0.001 / 1030</td> <td>0.6</td> <td>0.00017 / 170</td> <td>0.75</td> <td>2.5e-05 / 25</td> </tr>
</table>

The threshold values are rounded to the nearest 0.05.


# Method

I won't get into heavy detail here, the actual notebooks are linked below.

## Similarity between random molecules

The workflow and dataset for this is described in a [blog
post](https://greglandrum.github.io/rdkit-blog/fingerprints/similarity/reference/2021/05/18/fingerprint-thresholds1.html).
The very quick summary is that I generated statistics for the similarity
distribution of 25K random pairs of reasonable sized (MW<600) molecules exported
from ChEMBL.

## Groups of related compounds

This is really the central pillar of the post: how do we pick sets of compounds
we can use to quantify similarity search performance? 

One obvious possibility is to just take groups of molecules which are known to
be active against the same targets. This is the classic similarity-based virtual
screening use case and it's one which has been done a lot in the literature.
That's an interesting (and important) use case and it's something which I may
come back to in a future post, but it requires a connection between chemical
similarity and biological activity. That connection (or lack thereof) makes
analysis of the threshold results more complex and introduces a significant amount of
variability. 

Here I want to look at a different use case: searching a database and retrieving
compounds which are chemically similar to each other. For this I need to pick
groups of chemically similar compounds without actually using a traditional
approach to chemical similarity. The approach I used is to assume that the
typical medicinal chemistry SAR paper includes a bunch of compounds which come from
a small number of chemical series (typically one). These compounds are
definitely related to each other and it's not unreasonable to expect that a
similarity search for one should return the others as results.

This led me back to an earlier [blog post looking at identifying scaffolds from
ChEMBL compounds tested in the same
assay](http://rdkit.blogspot.com/2015/05/a-set-of-scaffolds-from-chembl-papers.html)
(given the structure of ChEMBL, this implies that the compounds are from the
same paper). That post includes some pre-filtering of the results to try and get
only SAR papers by only keeping assays (papers) where 50-100 compounds were
measured. For this post I re-ran that analysis against ChEMBL28. The analysis
produced results for 402 groups (a group is the compounds tested in one assay);
for this analysis I further filtered these down to the 301 groups (20342
compounds in total) where the number of atoms in the scaffold is at least 50% of
the average number of atoms for compounds in the group.

So given these 301 groups of chemically related compounds I was ready to start
doing some searches.

## Determining background retrieval rates

In order to get a sense of how many compounds would be retrieved from a database
when using the related compounds, I randomly picked 100K molecules from ChEMBL28
to use as a background. I wanted a representative sample, so I didn't apply MW
filters when doing this selection.

I then queried the background compounds with each molecule in a random subset of
the 20K members of the "related compounds" set, counted the number of results
each returned for each fingerprint/similarity threshold combination, and did
statistics based on those results.


## Summarizing the data

Here's an example of a graphical summary of the results presented in the final
notebook listed below:

![]({{ site.baseurl }}/images/blog/similarity-search-threshold1-img1.png)

The violin plots show the distribution of similarity values required to match
50% of the related compound pairs for each of the fingerprints. The dark gray
boxes show the noise level for the fingerprints. The red line shows the median
fraction of the 100K ChEMBL compounds retrieved when using the median value from
the violin plots as a similarity threshold.


## The notebooks

Here are the github links for the notebooks I used:
- Similarity between random molecules (this is the previous analysis): [https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Fingerprint%20Thresholds.ipynb](https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Fingerprint%20Thresholds.ipynb)
- Finding scaffolds for ChEMBL documents (also a previous analysis): [https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Finding%20Scaffolds%20Revisited%20again.ipynb](https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Finding%20Scaffolds%20Revisited%20again.ipynb) 
- Similarity distributions for related compounds: [https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Fingerprint%20Thresholds%20Scaffolds.ipynb](https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Fingerprint%20Thresholds%20Scaffolds.ipynb)  Note that this is a new one and I'm still working on cleaning it up and adding more text/explanation
- Fraction of the database retrieved when searching (this one also has the calculation of the summary results presented here): [https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Fingerprint%20Thresholds%20Database%20Fraction.ipynb](https://github.com/greglandrum/rdkit_blog/blob/master/notebooks/Fingerprint%20Thresholds%20Database%20Fraction.ipynb) Note that this is a new one and I'm still working on cleaning it up and adding more text/explanation


