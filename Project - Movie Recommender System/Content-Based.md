3. Analyzing Content
4. Profiling :
	1. Unsupervised Feature Selection : We don't look at the ratings when removing. 
	2. Supervised : We look at the ratings to choose which features to keep/remove. 
		- We use Khi2-stat : We try to find if the rating has an impact on the user's choice. 
		- Example with Netflix : 1/10 movies are seen by user, 2/10 of films are Drama ; We compare theoretical data (How it is supposed to be) and then compare it with the reality using khi2 (for unary and binary data). If both are very different, value of khi2 is high. 
		- Gini (for binary and discrete) : It says how are the values distributed, if it has more variation it's probably more likely important. 
		- Dev : give also some distribution
		- ML type feature selection
			- incremental : assumes to have several features (5) and want to choose some of them (most relevant). MSE test on only 1,2,3,4,5. Then test them in every combination possible by adding each to the best one and see which comes up the best.
			- decremental : start from everything and remove everytime the worst one.
	3. 