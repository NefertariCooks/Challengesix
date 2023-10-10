# Challengesix
This is main portion of the code from Challenge 6

// Challenge 6: Break repeating-key XOR
#include <iostream>
#include <vector>
#include <fstream>
#include <string>

using namespace std;

//base64 characters
const string BASE_64 = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

//base 64 decoder
vector<unsigned char> b64Decode(string);
//single-character XOR, XOR's every byte in the input buffer against a single character
string xorChar(vector<unsigned char>, unsigned char);
//english plaintext scoring function
double englishScore(string);
//hamming distance calculator
int calcHammingDistance(string, string);
//key size guesser 
void guessKeySize(vector<unsigned char>, vector<int>& bestKeySizes);

int main()
{
    //open input file 
    ifstream inFile;
    inFile.open("cryptopals.com_static_challenge-data_6.txt");
    if (!inFile.is_open())
    {
        cout << "Error opening input file. Ensure it is in the correct directory." << endl;
        return -1;
    }

    //get input from file
    string base64Text = "";
    string oneLine = "";
    while (getline(inFile, oneLine))
    {
        //concatenate all base64 text into one long string
        base64Text += oneLine;
    }
    inFile.close();

    //decode the base64 input
    vector<unsigned char> bytes = b64Decode(base64Text);

    //find the three best cipher key lengths
    vector<int> bestKeySizes;
    guessKeySize(bytes, bestKeySizes);

    string ciphers[3] = { "", "", "" };

    //loop through the three best key sizes
    for (int currKey = 0; currKey < 3; currKey++)
    {
        //current key size
        int currSize = bestKeySizes[currKey];

        //store matched blocks
        vector<string> matchedBlocks;

        //transpose blocks into matching blocks based on key size
        for (int i = 0; i < currSize; i++)
        {
            string mBlock = "";
            for (int j = i; j < bytes.size(); j += currSize)
            {
                mBlock += bytes[j];
            }
            //all bytes are now organized based on position for the repeating-key cipher
            matchedBlocks.push_back(mBlock);
        }

        //find the current repeating xor key
        string repXorKey;

        //brute force test each block against a character and score the resulting plaintext
        for (int i = 0; i < matchedBlocks.size(); i++)
        {
            //variables for testing
            double hiScore = 0;
            double currScore = 0;
            unsigned char bestKey = ' ';
            string currEnglish;            
            vector<unsigned char> blockText;
            string currBlock = matchedBlocks[i];    

            for (int j = 0; j < currBlock.length(); j++)
            {
                blockText.push_back(currBlock[j]);
            }

            //test every single character
            for (unsigned int j = 0; j < 256; j++)
            {
                unsigned char currKey = j;
                currEnglish = xorChar(blockText, currKey);
                currScore = englishScore(currEnglish);

                //track best score
                if (currScore > hiScore)
                {
                    hiScore = currScore;
                    bestKey = currKey;
                }

            }

            //append to the end of the best xor key
            repXorKey += bestKey;

        }
        ciphers[currKey] = repXorKey;
        repXorKey = "";
    }

    //test each key and decrypt the message in repeating-key xor manner
    unsigned int position = 0;      //position in key
    string testOutput = "";         //test output
    string output = "";             //actual output to be displayed
    string bestCipher = "";         //the repeating-key cipher used
    double hiScore = 0;             //best english score
    double currScore = 0;           //current english score

    //loop through the 3 potential cipher keys
    for (int i = 0; i < 3; i++)
    {
        //reset position and testOutput
        position = 0;
        testOutput = "";

        //decrypt
        for (int j = 0; j < bytes.size(); j++)
        {
            unsigned char byte1 = bytes[j];
            unsigned char byte2 = ciphers[i][position];
            unsigned char xorByte = (byte1 ^ byte2);
            testOutput += xorByte;

            if (position == (ciphers[i].length() - 1))
            {
                position = 0;
            }
            else
            {
                position++;
            }
        }

        //track best one
        currScore = englishScore(testOutput);
        if (currScore > hiScore)
        {
            hiScore = currScore;
            output = testOutput;
            bestCipher = ciphers[i];
        }
    }

    //output the decrypted text
    cout << "Challenge 6: Detect Repeating-Key XOR" << endl;
    cout << "Repeating-Key cipher used: " << endl << bestCipher << endl << endl;
    cout << "Decrpyted message is long. Press enter to display it." << endl;
    cin.get();
    cout << output;


    return 0;
}
