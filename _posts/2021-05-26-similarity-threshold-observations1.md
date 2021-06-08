---
toc: true
layout: post
description: Fingerprint efficiency
categories: [similarity, reference]
title: Some observations about similarity search thresholds
image: images/blog/similarity-search-observations1-img1.png
---

**Updated 08.06.2021** after I expanded the set of "related compounds". The source
of the previous version of the post is available [in
github.](https://github.com/greglandrum/rdkit-blog/blob/c5eb92294c45e7fe77b6c5658d1de5388090d7f1/_posts/2021-05-26-similarity-threshold-observations1.md) The updates didn't change the discussion that much.


# TL;DR

Based on the analysis here it looks like the fingerprint the RDKit provides
which does the best job of efficiently retrieving chemically similar structures
is the RDKit fingerprint with `maxPath` set to 6.

# Intro / Results

I recently did a post presenting an [approach for finding reasonable
thresholds](https://greglandrum.github.io/rdkit-blog/similarity/reference/2021/05/21/similarity-search-thresholds.html)
for similarity searches using the fingerprints the RDKit provides. This is a
followup to that one written after I've done some more looking at the data. I
want to come up with a suggestion for which fingerprint to use for similarity
searches when the goal is retrieving as many chemically related compounds as
possible. I'll do that by looking at search efficiency as measured by the
fraction of the total database retrieved when using similarity thresholds
sufficient to return 90-95% of the related compounds. See the [earlier
post](https://greglandrum.github.io/rdkit-blog/similarity/reference/2021/05/21/similarity-search-thresholds.html)
for an explanation of what "related compounds" means here and how the searches
were done.

As a reminder, this is how I presented the results in that post and how to interpret the data:

<table>
<tr><th></th> <th></th> <th colspan="2">0.95 of related compounds</th> <th colspan="2">0.9 of related compounds</th> <th colspan="2">0.8 of related compounds</th> <th colspan="2">0.5 of related compounds</th></tr>
<tr><th>Fingerprint</th> <th>0.95 noise level</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th></tr>
<tr>
<td><b>Morgan2 (bits)</b></td> <td>0.27</td> <td>0.4</td> <td>0.00019 / 190</td> <td>0.4</td> <td>0.00019 / 190</td> <td>0.45</td> <td>0.00012 / 115</td> <td>0.55</td> <td>2.5e-05 / 25</td> </tr>
</table>

The 0.95 noise level (from the previous analysis) for the MFP2 fingerprint is
0.27. If I want to retrieve 95% of the related compounds I need to set the
similarity threshold to 0.4. With this threshold I would retrieve ~190
compounds per million compounds in the database (0.4% of the database).
Similarly, if I were willing to live with finding 50% of the related actives I
could set the search threshold to 0.55, in which case I'd only retrieve ~25 rows
per million compounds in the database.

I won't reproduce the full results table from the post here, but here are the
rows with the highest search efficiencies (lowest number of compounds returned
from the "background database") at 90% and 95% of related compounds found. I
sorted the table by the efficiency at 90% of related compounds retrieved:

<table>
<tr><th></th> <th></th> <th colspan="2">0.95 of related compounds</th> <th colspan="2">0.9 of related compounds</th> <th colspan="2">0.8 of related compounds</th> <th colspan="2">0.5 of related compounds</th></tr>
<tr><th>Fingerprint</th> <th>0.95 noise level</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th> <th>threshold</th> <th>db fraction / count per million</th></tr>
<tr>
<td><b>RDKit 7 (bits)</b></td> <td>0.43</td> <td>0.55</td> <td>0.00051 / 510</td> <td>0.6</td> <td>8e-05 / 80</td> <td>0.6</td> <td>8e-05 / 80</td> <td>0.7</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>Topological Torsions (counts)</b></td> <td>0.19</td> <td>0.35</td> <td>0.00049 / 489</td> <td>0.4</td> <td>0.00011 / 110</td> <td>0.45</td> <td>7.5e-05 / 75</td> <td>0.55</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>linear RDKit 7 (bits)</b></td> <td>0.26</td> <td>0.45</td> <td>0.00053 / 535</td> <td>0.5</td> <td>0.00013 / 130</td> <td>0.55</td> <td>9e-05 / 90</td> <td>0.65</td> <td>3.5e-05 / 35</td> </tr>
<tr>
<td><b>RDKit 6 (bits)</b></td> <td>0.31</td> <td>0.5</td> <td>0.00021 / 210</td> <td>0.55</td> <td>0.00014 / 135</td> <td>0.6</td> <td>6e-05 / 60</td> <td>0.7</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>Morgan2 (counts)</b></td> <td>0.25</td> <td>0.4</td> <td>0.00014 / 140</td> <td>0.4</td> <td>0.00014 / 140</td> <td>0.45</td> <td>8.5e-05 / 84</td> <td>0.55</td> <td>2e-05 / 20</td> </tr>
<tr>
<td><b>Avalon 1024 (bits)</b></td> <td>0.37</td> <td>0.55</td> <td>0.00075 / 750</td> <td>0.6</td> <td>0.00014 / 140</td> <td>0.65</td> <td>9e-05 / 90</td> <td>0.75</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>Morgan3 (counts)</b></td> <td>0.20</td> <td>0.3</td> <td>0.00026 / 260</td> <td>0.35</td> <td>0.00015 / 154</td> <td>0.35</td> <td>0.00015 / 154</td> <td>0.45</td> <td>3.5e-05 / 35</td> </tr>
<tr>
<td><b>RDKit 5 (bits)</b></td> <td>0.29</td> <td>0.5</td> <td>0.00025 / 250</td> <td>0.55</td> <td>0.00016 / 155</td> <td>0.6</td> <td>6e-05 / 60</td> <td>0.7</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>Topological Torsions (bits)</b></td> <td>0.22</td> <td>0.4</td> <td>0.00016 / 160</td> <td>0.4</td> <td>0.00016 / 160</td> <td>0.45</td> <td>0.00011 / 105</td> <td>0.55</td> <td>3.5e-05 / 35</td> </tr>
<tr>
<td><b>Morgan2 (bits)</b></td> <td>0.27</td> <td>0.4</td> <td>0.00019 / 190</td> <td>0.4</td> <td>0.00019 / 190</td> <td>0.45</td> <td>0.00012 / 115</td> <td>0.55</td> <td>2.5e-05 / 25</td> </tr>
<tr>
<td><b>FeatMorgan3 (counts)</b></td> <td>0.28</td> <td>0.4</td> <td>0.00022 / 220</td> <td>0.4</td> <td>0.00022 / 220</td> <td>0.45</td> <td>0.00013 / 130</td> <td>0.55</td> <td>3e-05 / 30</td> </tr>
<tr>
<td><b>linear RDKit 6 (bits)</b></td> <td>0.28</td> <td>0.5</td> <td>0.00022 / 220</td> <td>0.5</td> <td>0.00022 / 220</td> <td>0.55</td> <td>0.00014 / 140</td> <td>0.7</td> <td>3e-05 / 30</td> </tr>
</table>
The threshold values are rounded to the nearest 0.05.

I've included count-based fingerprints in the above table, but they wouldn't be
my first choice for use in a real-world similarity search application.
Calculating similarity for count-based fingerprints is significantly slower than
bit vector fingerprints, so they really aren't practical for large datasets.
Note that the RDKit has a method for approximating counts using bit vector
fingerprints which is used by the Atom Pair and Topological Torsion fingeprints
and could also be an option for the other fingerprint types, but that's a topic
for another post.

Based on these numbers (and, of course, the dataset I used) it looks like the
RDKit fingerprint is the optimal choice for chemical similarity search. Taking
the efficiency at both 90% and 95% into account, the version of the fingerprint
with `maxPath=6` is arguably better than the version with `maxPath=7` (which is
the default). There's not a publication for the RDKit fingerprint but it is
[described in detail in the RDKit
documentation](https://www.rdkit.org/docs/RDKit_Book.html#rdkit-fingerprints).

The Morgan3 fingerprint, which is what I kind of expected to be the best at this
task, doesn't do that well - the bit-vector based form didn't even make this
list of top performaers. The Morgan2 fingerprint, on the other hand, seems like
another good choice. The Morgan fingerprints are the RDKit's implementation of
the circular fingerprints described in [this
publication](https://doi.org/10.1021/ci100050t).

A real surprise to me was how well the topological torsions fingerprint does at
this chemical search. I had (I guess without much evidence) thought of it as
more of a fuzzy (or "scaffold-hopping") fingerprint, but the high efficiency on
this chemical search problem makes me reconsider that. Topological torsions were
introduced in [this publication](https://doi.org/10.1021/ci00054a008).

The Avalon fingerprint seems to be another decent choice, at least at 90%. This
isn't surprising to me, but I'll probably remain resistant to making heavy of it
due to the complexity of the fingerprint itself. The only non-code description
I'm aware of for the Avalon FP is in the supplementary material for [this
paper](https://pubs.acs.org/doi/10.1021/ci050413p); it's likely that the current
version of the fingerprint, which was under active development for at least 10
years after that paper appeared, deviates from that.

Before getting any deeper into details with this kind of analysis, I think I
would like to look into using more than 10K of the "related" molecules and
increasing the size of the background database just to make sure the statistics
are solid. I'll do that in a separate post and leave the count-based
fingerprints out.


