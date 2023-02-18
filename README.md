# CSE508_Winter2023_A1_69
 Information Retrieval Assignment 1, IIITD

The code notebook might not be viewable in github as it is a pretty big file. Please download it in your system or open it with Google Colab to view. 
Q1.  Data Pre-processing
Methodology:
1.	Relevant libraries are imported
2.	The path of the working directory where dataset lies in the local machine is set, and the working directory is set to that location with ‘os’ library (os.chdir())
3.	Relevant text is extracted between <TITLE></TITLE> and <TEXT></TEXT> tags and the retrieved text is overwritten in the same files.
4.	Preprocessing functions are defined and applied on each document and a final preprocessed dataset is stored in a list ‘final_dataset’

Preprocessing Steps:
1.	The text of each document is lowercased
2.	The text of each document in converted into tokens, each token representing one word 
3.	Stopwords are removed from tokens
4.	Punctuations are removed from tokens. Any pattern like [^\w\s] is replaced with whitespaces ‘ ’
5.	Blank space tokens are removed

Results: <Results can be viewed in notebook>
 
Assumptions:
Processed data is written back to the same file only is Q1 (i)
For Q1 (ii), after each pre-processing step, the processed data is not written back to the file rather is stored in a separate list.
-----------------------------------------------------------------------------------------------------------------------------------------------------------
Q2. Unigram Inverted Index
Methodology:
Creating the unigram inverted index
1.	An empty dictionary is created to store unigram inverted indexes
2.	Each document token list in the preprocessed corpus dataset is iterated
3.	Tokens for each document are created in each iteration
4.	For each token created, if that token already exists in dictionary, then corresponding document is added in the value component of that token in the dictionary
5.	If token does not exist in the dictionary, then a new key-value pair (token-document) is added in the dictionary
6.	Unigram index dictionary is dumped in local directory using ‘pickle’ and loaded back
Boolean query processing
1.	All the pre-processing steps of Q1 are applied to input query by calling appropriate functions sequentially
2.	A get_not() function is defined which retrieves all the documents in the corpus that does not contain the passed word. Here comparisons will be equal to total documents in the corpus.
3.	A generate_operators_operands() function separates the input query words and operators (and, or) in separate lists
4.	A set intersection() and set union() functions are defined to take intersection and union of two passed sets. Here comparisons will be equal to number of documents in set1.  (Not optimized for iterating the set with less number of documents)
5.	The and_or() function takes the intersection or union of the postings set of each query token in a sequential manner
6.	The run_query() function processes all ‘not’ operators first and stores their corresponding returned sets in separate list ‘not_set_list’ before taking intersection or union depending on ‘and’ or ‘or’ operation.
7.	Note that each term of the type ‘NOT term’ is first processed and corresponding not set index in not_set_list is passed in operand in place of ‘not term’

Assumptions:
1.	The number of tokens present in the processed query tokens list must be exactly 1 more than the number of operators passed for that query.
2.	This does not apply for input query containing only one token and one not operator. In this case, program automatically returns appropriate document set for given query of type ‘NOT token’
3.	If point 1 is not satisfied for queries of type other than the type mentioned in point 2, then the program will automatically scale down the number of tokens if not enough operators are provided or it will scale down number of operators if not enough tokens are available to apply those operators on. Scaling down here means, the extra operands or operators will be ignored

Results: <Results can be viewed in notebook>
-----------------------------------------------------------------------------------------------------------------------------------------------------------
Q3. Bigram Inverted Index and Positional Index

Methodology:
(i)	Creating the Bigram inverted index
1.	An empty dictionary is created to store bigram inverted indexes
2.	Each document token list in the preprocessed corpus dataset is iterated
3.	Tokens for each document are created in each iteration
4.	For each ith token, a biword is created in the form “token[i]+‘ ’+token[i+1]” until the (n-1)th token where n is number of tokens in current document
5.	If that biword already exists in dictionary, then corresponding document is added in the value component (posting) of that byword in the dictionary
6.	If biword does not exist in the dictionary, then a new key-value pair (biword-document) is added in the dictionary
7.	Bigram index dictionary is dumped in local directory using ‘pickle’ and loaded back
(ii)	Creating the Positional index
1.	An empty dictionary is created to store positional indexes
2.	Each document token list in the preprocessed corpus dataset is iterated
3.	Tokens for each document are created in each iteration and each token is iterated
4.	If that token is positional index dictionary and also its docID is in 
5.	If that biword already exists in dictionary and also its docID is in postings list of that token, then we update the token position in postings list of docID for that token
6.	If token exists but its docID does not, then new docID is added in postings list of that token along with its position
7.	If token is not present at all in the dictionary, then that token with its docID and position is added in the positional index dictionary
8.	Final positional index dictionary is dumped in the local folder and loaded back using ‘pickle’

(iii)	Boolean query processing
1.	All the pre-processing steps of Q1 are applied to input query by calling appropriate functions sequentially
2.	A run_query_bigram() function iterates through all input query tokens (sanitized) and creates a biword for each ith and (i+1)th token. 
3.	In initial iteration, the result set consists the postings list of the very first biword. From the 2nd iteration onwards, the intersection of postings list set of current biword and previous result set is taken and stored back in result set variable. In this way, result set after last iteration is out required set of documents.
4.	The helper function positional_find_matching_documents(tokens) finds all the documents in which all passed query tokens are present.
5.	The helper function positional_intersect_for_token_pair(token1, token2, distance) returns the set of documents in which token2 is farther than ‘distance’ words from the token1
6.	A run_query_positional(query_tokens) function first retrieves the matching documents i.e. the documents in which all input query tokens (sanitized) are present. Is this matching documents sets itself is empty then no need to look further as our resulting set will be empty set (no need to check relative positions of tokens)
7.	If matching documents set is not empty, then we check for documents satisfying relatives positions of each pair of tokens, by intersecting the document set returned from positional_intersect_for_token_pair() for each token pair with previous result set and storing the result back in result set.
8.	We consider each token pair by nested for loop where,
i goes from 0 to len(query_tokens) and j goes from (i+1) to len(query_tokens)
For each such iteration, the distance to check between two tokens will be,
|(j-1)|

Assumptions:
1.	The document containing text : “…token1 stopword1 stopword2 stopword3 token2…” is considered valid for phrase query “token1 stopword4 stopword5 token2”. These cases won’t count in False Positives as we perform preprocessing on documents as well as on the input query. Thus stopwords won’t make any impact.
2.	The relative positions of tokens with each other are taken into consideration only after pre-processing the input query and NOT before, as we need to apply these token positions on an already pre-processed data 

Results: <Results can be viewed in notebook>
 
As we can see from above query inputs, For query 1 - 'transition supersonic wind', the bigram inverted index shows 4 documents which says that 'transition supersonic wind' is present in these 4 documents. Although these words do appear in these documents but they do not appear in the specified sequence and adjacent to each other. Hence Bigram inverted index shows False Positives for our query. Positional index retrieves 0 documents for the same query. Thus positional index is a better representation of document indexes for queries containing more than 2 words as Positional index also takes into account the relative positions of the input query words and Bigram index doesn't.
Also, in query 2 - 'transition reynolds numbers', the bigram inverted index shows 7 documents while positional index shows just 5 documents. The document set shown by positional index is the true set as 'transition reynolds numbers' appear in the exact given sequence in only these 5 documents (ignoring the stopwords). Thus Bigram index shows two false positives in this case. Thus we can conclude that, the number of documents retrieved by Positional index will always be less than or equal to number of documents retrieved by Bigram inverted index. Bigram inverted index will often show False Positives for queries with more than 2 tokens
On the other hand, Bigram index and Positional index shows same result for query 3 - 'experimental investigation'. This is because for a 2-word input query (after removing stopwords), Bigram index and Positional index behaves same. Not just for this particular input but for any input query containing only 2 tokens, the Bigram and Positional index will always show the same result as a 2 word query in positional index just behaves same a biword behaves in bigram index. In Positional index, the fact that position of token2 is 1 ahead of position of token1 is considered. And bigram index is constructed on basis of this very same assumption. Thus for a 2-word query searching, bigram inverted index and positional index will always show same result.
