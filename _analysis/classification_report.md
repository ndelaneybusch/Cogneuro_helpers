---
title: Informative Classification Reports
---
# Description
This function takes a classifier's predicted labels and the actual labels and returns a useful report of classification metrics. Accepts label identities, one-hot encoding class arrays, or class probabilities, and returns the same metrics regardless of input format (no munging required). Output includes !) a confusion matrix (with an "expand" option to prevent wrapping when printing to console), totals for false positivies and false negatives, B) a table of precision/recall/F1 values, C) accuracy and cohen's kappa value, and D) sensitivity and specificity. 

# Dependencies
{% highlight python %}
import numpy as np
import pandas as pd
from sklearn import metrics
{% endhighlight %}

# Function
{% highlight python %}
def my_classification_report(test_predictions, test_labels, expand=True):
    if len(test_predictions[0].shape)==1:
        c = b.argmax(axis=-1)
    else:
        c = test_predictions
    if len(test_labels[0].shape)==1:
        test_labels = test_labels.argmax(axis=-1)
    confusion_matrix = metrics.confusion_matrix(test_labels, c)
    FP = confusion_matrix.sum(axis=0, dtype=float) - np.diag(confusion_matrix)  
    FN = confusion_matrix.sum(axis=1, dtype=float) - np.diag(confusion_matrix)
    TP = np.diag(confusion_matrix)
    TN = confusion_matrix.sum() - (FP + FN + TP)
    confusion_matrix_table = pd.crosstab(test_labels, c, rownames=['True'], colnames=['Predicted'], margins=True)
    confusion_matrix_table['False Negatives']=np.append(FN, sum(FN))
    FProw = pd.DataFrame(FP).transpose()
    FProw['All']=sum(FP)
    confusion_matrix_table = confusion_matrix_table.append(FProw.rename({0: "False Positives"}))
    if expand==True:
        pd.set_option('expand_frame_repr', False)
    print("Confusion Matrix (predicted across columns, actual down rows)")
    print(confusion_matrix_table)
    print("")
    print("Classification Report")
    print(metrics.classification_report(test_labels, c))
    print("")
    print("accuracy: "+str(metrics.accuracy_score(test_labels, c)))
    print("cohen's kappa: "+str(metrics.cohen_kappa_score(test_labels, c)))
    print("sensitivity: "+str((TP/(TP+FN)).round(4))) #sensitivity
    print("specificity: "+str((TN/(TN+FP)).round(4))) #specificity
    pd.reset_option('expand_frame_repr')
    return
{% endhighlight %}

# Function Call
{% highlight python %}
my_classification_report(test_predictions, test_labels) %automaticall expands tables to prevent wrapping
my_classification_report(test_predictions, test_labels, expand=False) %wraps tables if they are too long
{% endhighlight %}

__Sample output:__
From a quick classifier on the <a href='https://ndelaneybusch.github.io/Cogneuro_helpers/2017-09-26-notmnist-inception/'>notMNIST</a> 10-class problem.
<pre>
Confusion Matrix (predicted across columns, actual down rows)
                     0       1       2       3      4      5       6      7      8       9      All  False Negatives
0                956.0     4.0     2.0    13.0    3.0    1.0     5.0    9.0    5.0     2.0   1000.0             44.0
1                  1.0   978.0     4.0     4.0    2.0    1.0     7.0    0.0    0.0     3.0   1000.0             22.0
2                  1.0     1.0   979.0     4.0    2.0    0.0    12.0    0.0    0.0     1.0   1000.0             21.0
3                  2.0     6.0     3.0   975.0    0.0    0.0     3.0    0.0    2.0     9.0   1000.0             25.0
4                  1.0     7.0    30.0     4.0  937.0    2.0     9.0    0.0    6.0     4.0   1000.0             63.0
5                 10.0     7.0     2.0     5.0    7.0  936.0     8.0    4.0    6.0    15.0   1000.0             64.0
6                  0.0     4.0    22.0     4.0    4.0    0.0   963.0    0.0    1.0     2.0   1000.0             37.0
7                 11.0     7.0     0.0     8.0    0.0    0.0     6.0  953.0    5.0    10.0   1000.0             47.0
8                  1.0    13.0     5.0    20.0    3.0    1.0     6.0    1.0  904.0    46.0   1000.0             96.0
9                  3.0     2.0     1.0    10.0    2.0    0.0     5.0    0.0    7.0   970.0   1000.0             30.0
All              986.0  1029.0  1048.0  1047.0  960.0  941.0  1024.0  967.0  936.0  1062.0  10000.0            449.0
False Positives   30.0    51.0    69.0    72.0   23.0    5.0    61.0   14.0   32.0    92.0    449.0              NaN

Classification Report
             precision    recall  f1-score   support

          0       0.97      0.96      0.96      1000
          1       0.95      0.98      0.96      1000
          2       0.93      0.98      0.96      1000
          3       0.93      0.97      0.95      1000
          4       0.98      0.94      0.96      1000
          5       0.99      0.94      0.96      1000
          6       0.94      0.96      0.95      1000
          7       0.99      0.95      0.97      1000
          8       0.97      0.90      0.93      1000
          9       0.91      0.97      0.94      1000

avg / total       0.96      0.96      0.96     10000


accuracy: 0.9551
cohen's kappa: 0.950111111111
sensitivity: [ 0.956  0.978  0.979  0.975  0.937  0.936  0.963  0.953  0.904  0.97 ]
specificity: [ 0.9967  0.9943  0.9923  0.992   0.9974  0.9994  0.9932  0.9984  0.9964
  0.9898]
  </pre>
