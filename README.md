# SIMON: A Simple Measurement Ontology
SIMON: SImple Measurement ONtology is designed to represent measurements of one, two or n dimensions. E.g., distance: meters, mass: kilograms, time: seconds; speed: miles per hour; acceleration: meters per second per second. Although there are other vocabularies that exist in this domain they are large and complex. That is of course completely appropriate for certain types of problems but for many problems I've encountered recently a smaller, leaner, easier vocabulary is useful, that is the niche that SIMON attempts to fill. 

There are two versions of the ontology: SIMON and SIMON Processed. The processed version of the ontology has all the SWRL inferences already exported using the Protege command Export Inferred Axioms as Ontology. Also, the processed version is the result of my running the SPARQL transformations (see the SPARQL file) that automatically format the rdfs:label of each measurement. Running these SPARQL transforms  saves the user the effort and potential errors of entering a nicely formatted name for each measurement. The SPARQL transforms use the rdfs:label values and the hasMeasuredValue value to construct a string such as "5 Miles per Hour" and make that the label of the MPH5 instance. Note: there is another data property on the unit instances called pluralName. This is used to construct the names because the first unit needs to be plural rather than singular which is the typical standard for class names that SIMON follows. 

There are two versions of the SPARQL transforms (both versions are in the SPARQL file). One version uses CONSTRUCT to create the labels. This can be used with the Snap SPARQL plugin for Protege. The other transforms use the INSERT clause. This requires you load the ontology into some environment other than Protege since as of now neither of the SPARQL implementations in Protege support INSERT or DELETE. 

There are only three top level classes: Measurement, Unit, and PunnedClass. The Unit class has subclasses: DistanceUnit, TimeUnit, and MassUnit. The instances of Unit are individuals such as Mile, Hour, and Kilogram. Of course this can easily be extended with additional units such as Euros for currency. The Measurement class has subclasses SingleUnitMeasurement, DoubleUnitMeasurement, and TripleUnitMeasurement for measurements in one, two and three dimensions respectively. E.g., instances of SingleUnitMeasurement are Miles10, Hours1, and Kilograms20. Example instances of DoubleUnitMeasurement and TripleUnitMeasurement are Mph20 (speed) and Kpsps10 (acceleration). I put the number after the name of each individual so that in a list of individuals all measurements with the same units will be viewed together. The design also accomodates extending the ontology with 4 or more dimensions. This is an example of the design pattern for transforming N-ary relations to binary relations: https://www.w3.org/TR/swbp-n-aryRelations/ 

The Measurement class has three data properties: hasMeasuredValue is an xsd:decimal with the actual value and measuredAtTime is an xsd:dateTime that optionally records the time the measurement was taken. Finally, hasCanonicalValue has range of xsd:decimal and is the value of the measurement when it is transformed into the canonical value for that type of measurement. The PunnedClass class is used for instances of punned subclasses of the Measurement class. This is used so that the canonical unit for each measurement can be stored there using the hasCanonicalUnit object property. 

Other object properties are hasUnit1, hasUnit2, and hasUnit3 which define the Unit for each Measurement. The other properties are greaterThan, lessThan, greaterThanOrEqualTo, equalTo, etc. These define an order for Measurement individuals that have the same number of dimensions and kind of units. The greaterThan and lessThan properties are inverses and transitive. Because the properties are transitive it is only required to define an order in one direction and the inverse property will automatically be inferred. However, it is still necessary to test for greaterThan and lessThan otherwise there is a risk that the maximum or minimum value will be excluded from the order. Thus I test for lessThan and greaterThanOrEqualTo with SWRL rules and let the inverses for each fill in the appropriate values. 

I have also defined SWRL rules that convert each Measurement to the canonical unit for that kind of measurement.  E.g., the canonical unit for distance is meters. The canonical unit for each type of measure is stored on the class itself by utilizing punning. For example, in the SWRL rules that convert different measurements such as inches and miles to meters there is the expression: 

hasCanonicalUnit(DistanceMeasurement, Meter)

I.e., these rules will only fire if the hasCanonicalUnit for the DistanceMeasurement class is equal to the instance of unit: Meter. To change the canonical unit one merely needs to:

1) Change the value on the punned individual for the measurement class to the new canonical unit. E.g., if you wanted to change the canonical unit for distance to kilometers, you would change the value from Meter to Kilometer

2) Clone each of the relevant existing SWRL rules and change the expression to test for the new canonical unit. You could also simply edit each rule but I would suggest cloning them. This way you still have the old rules if you decide to go back to that canonical unit. 

3) Change the conversion factor to the appropriate number in each of the cloned or edited rules. 

For example, the current rule to transform FeetMeasurement to meters is:

hasCanonicalUnit(DistanceMeasurement, Meter) ^ DistanceMeasurement(?m1) ^ hasUnit1(?m1, Foot) ^ hasMeasuredValue(?m1, ?v1)   ^ swrlb:multiply(?cv, ?v1, 0.3048) -> sm:hasCanonicalValue(?m1, ?cv)

To change the canonical unit to kilometers one would edit the hasCanonicalUnit value for DistanceMeasurement, clone this rule, and then edit the cloned rule as follows:

hasCanonicalUnit(DistanceMeasurement, Kilometer) ^ DistanceMeasurement(?m1) ^ hasUnit1(?m1, Foot) ^ hasMeasuredValue(?m1, ?v1)   ^ swrlb:multiply(?cv, ?v1, 0.0003048) -> sm:hasCanonicalValue(?m1, ?cv)

The canonical unit doesn't matter, the results are the same. The only reason to change it is if your ontology represents primarily very large or very small measurements you might want to change the canonical unit to reduce rounding errors. Note that originally the equalTo object property was symmetric however OWL reasoners infer that the domain and range of any symmetric property must be owl:Thing which caused problems (e.g., for Disjoint axioms). Since the equalTo test checks if each Measurement is equalTo itself the symmetric characteristic could be done away with. 
