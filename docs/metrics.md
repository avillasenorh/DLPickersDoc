# Metrics

Comparison of results from DL pickers with ground truth (high-quality picks).

True positives (\(t_p\)) are picks done by the Deep Learning picker that coincide
within a small time difference (\(\Delta t < \epsilon\)) with the picks from the
labelled dataset (e.g. refined IGN picks for El Hierro pre-eruption or CASTOR
sequence).

True negatives (\(t_n\))

Comparison metrics that we will use are:

$$ Precision = { t_p \over {t_p + f_p} } ,$$

$$ Recall = { t_p \over {t_p + f_n} } ,$$

<!--
$$ F_1 = { 2 \times {{Precision \times Recall} \over {Precision + Recall}} } $$

where \(t_p\) are true positives, \(f_p\) are false positives, and \(f_n\) are false negatives.
-->

