# "Python Data Science Handbook" by Jake Vanderplas

Page \#?
Use the following to print out an excellent confusion matrix of a classification algorithm result.  The trickiest part is the labels for x and y ticks.  Note that here y is simply a pandas series (from a larger DF).  The confusion matrix apparently takes all the unique classes you are trying to predict for given data (binary or multi-class) and sorts it alphabetically (or numerically, though I am unsure how it sorts when the classes you are predicting are string representations of numbers.  First guess is as a string.) So, the best way I found to handle this is to make y_name a sorted list of the unique classes you're predicting.  Also note certain datasets, such as those provided by SKLearn, have a `target_names` attribute which you can access and avoid this hassle.

```python
import seaborn as sns
print 'Confusion_matrix: '
# mat = confusion_matrix(y_test, y_pred)
mat = confusion_matrix(y, y)
y_name = sorted(y.unique())
ax = sns.heatmap(mat.T, square=True, annot=True, fmt='d', cbar=False, xticklabels=y_name, yticklabels=y_name, robust=True, vmax=None, vmin=None)

plt.xlabel('True Label')
plt.ylabel('Predicted Label')
plt.xticks(rotation='vertical')
plt.yticks(rotation='horizontal')
```
