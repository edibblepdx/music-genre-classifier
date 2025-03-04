> Classify music as blues, classical, country, disco, hip-hop, jazz, metal, pop, reggae, or rock

https://github.com/user-attachments/assets/c75d32ba-f4b9-417e-a1e5-ea8dcf3ffef4

I include along with an interface for the tool made to extract features, sample code for a DNN
and SVM, as well as the source for the website used to host my trained model. The method was to 
extract mel frequency capstone coeficients from the data for use in a Concurrent Neural Network 
(CNN) for preprocessing and feature reduction to be used as inputs to a Support Vector Machine 
(SVM) for classification. I was able to get 91.5% accuracy on the test set using an RBF Kernel. 
The SVM outperformed a Dense Nueral Network (DNN).

> [GTZAN Dataset](https://www.kaggle.com/datasets/andradaolteanu/gtzan-dataset-music-genre-classification)
>— I only use the raw audio and labels.

> app.py contains a flask app that when run in the browser will return a spectrogram image and genre prediction.

# Using the Feature Extractor

> I don't include my trained model in this repository

```python
from FeatureExtract import FeatureExtract
```
```python
fe = FeatureExtract()
x, y = fe.load_data('path/to/gtzan_wavs', 'path/to/gtzan_csv')
fe.train(features=x, labels=y, predict=False)
features = fe.extract(x) # returns a tf.Tensor
features = features.numpy() # to get a numpy array
```
### Subsequent use of the mfcc values can be made quicker
> These are not the values extracted from the CNN! Use these to train multiple CNNs.
```python
fe.save_csv('mfccs.csv', features, labels)
x, y = fe.load_csv('mfccs.csv')
```
### To save and load the CNN model after training
```python
fe.load_model(path)
fe.save_model(path=None, overwrite=False)
```
> You may then use this CNN feature extractor as input to some other classifier.

# Example use with SVM Classifier

```python
fe = FeatureExtract()
fe.load_model('cnn5.keras')
x, y = fe.load_csv('features.csv')

features = fe.extract(x).numpy()
print(features.shape)

label_encoder = LabelEncoder()
labels_encoded = label_encoder.fit_transform(y)

x_train, x_test, y_train, y_test = train_test_split(features, labels_encoded, test_size=0.2, random_state=42, stratify=labels_encoded)

clf = svm.SVC()
clf.fit(x_train, y_train)
predictions = clf.predict(x_test)

# confusion matrix
predicted_labels = label_encoder.inverse_transform(predictions)
true_labels = label_encoder.inverse_transform(y_test)
cm = confusion_matrix(true_labels, predicted_labels, labels=label_encoder.classes_)
display = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=label_encoder.classes_)
display.plot(cmap=plt.cm.Blues)
plt.xticks(rotation=45)
plt.title('Confusion Matrix')
plt.show()

# accuracy
accuracy = np.sum(predictions == y_test) / len(y_test)
print (f"accuracy: {accuracy}")
```


