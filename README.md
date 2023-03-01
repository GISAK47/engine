# engine
test
var image= ee.ImageCollection(ee.ImageCollection("LANDSAT/LE07/C02/T1_TOA"))
.filterBounds(ROI)
.filterDate('2011-01-01','2011-12-30')
.filterMetadata('CLOUD_COVER', 'less_than',3)
.mean()
.clip(ROI);
var locanh = image.focal_mean(1,'square', 'pixels',50)
var final = locanh.blend(image)
Map.addLayer(locanh,{bands:['B5','B4','B3']});

var label ='Class'
var bands =['B1','B2','B3','B4','B5','B7'];
var input = final.select(bands);
var training =(water).merge(urban).merge(thucvat).merge(dattrong);
print(training);

var trainImage = input.sampleRegions({
collection: training,
properties: [label],
scale: 30
});
print(trainImage);
var trainingData = trainImage.randomColumn();
var trainSet = trainingData.filter(ee.Filter.lessThan('random', 0.8));
var testSet = trainingData.filter(ee.Filter.greaterThanOrEquals('random', 0.8));
var classifier = ee.Classifier.smileCart().train(trainSet, label, bands);
var classified = input.classify(classifier);
print (classified.getInfo());

var landcoverPalette= [
  '#0D24DE', //0 water
  '#E20AF0', //1 urban
  '#147311', //2 thuc vat #31a354
  '#D3D30C', //3 dattrong
  ];
  Map.addLayer(classified, {palette: landcoverPalette, min: 0, max: 4}, 'classification');
 
 var confusionMatrix= ee.ConfusionMatrix(testSet.classify(classifier)
.errorMatrix({
  actual: 'Class',
  predicted: 'classification'
}));
print('Confusion Matrix:',confusionMatrix);
print("Overall Accuracy:",confusionMatrix.accuracy())
Export.image.toDrive({
  image: classified,
  description: 'Landsat_classified_CART_2011',
  scale: 30,
  region: image.geometry(),
  maxPixels: 1e13
})
