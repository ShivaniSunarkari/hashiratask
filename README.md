# hashiratask
#include <iostream>
#include <fstream>
#include <vector>
#include <map>
#include <set>
#include <cmath>
#include <algorithm>
#include <nlohmann/json.hpp>

using json = nlohmann::json;
using namespace std;

// Helper function: Compute LCM
long long lcm(long long a, long long b) {
    return abs(a * b) / __gcd(a, b);
}

// Helper function: Parse basic functions like sum, product, hcf, lcm
long long evaluateValue(const json& funcObj) {
    string func = funcObj["func"];
    vector<long long> args = funcObj["args"];
    if (func == "sum") {
        long long s = 0;
        for (auto val : args) s += val;
        return s;
    } else if (func == "product") {
        long long prod = 1;
        for (auto val : args) prod *= val;
        return prod;
    } else if (func == "hcf") {
        long long res = args[0];
        for (size_t i = 1; i < args.size(); ++i)
            res = __gcd(res, args[i]);
        return res;
    } else if (func == "lcm") {
        long long res = args[0];
        for (size_t i = 1; i < args.size(); ++i)
            res = lcm(res, args[i]);
        return res;
    }
    return 0;
}

// Lagrange interpolation to find constant term
long long lagrangeInterpolation(const vector<pair<long long, long long>>& shares) {
    long long secret = 0;
    int k = shares.size();

    for (int i = 0; i < k; ++i) {
        long long xi = shares[i].first;
        long long yi = shares[i].second;
        long long num = 1, den = 1;

        for (int j = 0; j < k; ++j) {
            if (i != j) {
                num *= -shares[j].first;
                den *= (xi - shares[j].first);
            }
        }
        secret += yi * (num / den); // Not safe for division — use modular inverse in real scenarios
    }
    return secret;
}

// Generate all K-combinations from shares
void generateCombinations(int index, int k, vector<pair<long long, long long>>& shares,
                          vector<pair<long long, long long>>& current,
                          vector<vector<pair<long long, long long>>>& combinations) {
    if (current.size() == k) {
        combinations.push_back(current);
        return;
    }
    if (index >= shares.size()) return;

    current.push_back(shares[index]);
    generateCombinations(index + 1, k, shares, current, combinations);
    current.pop_back();
    generateCombinations(index + 1, k, shares, current, combinations);
}

int main() {
    ifstream inFile("input.json");
    if (!inFile.is_open()) {
        cerr << "Could not open input file.\n";
        return 1;
    }

    json inputData;
    inFile >> inputData;

    int N = inputData["N"];
    int K = inputData["K"];
    vector<pair<long long, long long>> shares;

    for (auto& item : inputData["shares"]) {
        long long key = item["key"];
        long long value;

        if (item["value"].is_number()) {
            value = item["value"];
        } else {
            value = evaluateValue(item["value"]);
        }

        shares.emplace_back(key, value);
    }

    // Generate all K combinations
    vector<vector<pair<long long, long long>>> combinations;
    vector<pair<long long, long long>> current;
    generateCombinations(0, K, shares, current, combinations);

    map<long long, int> secretFrequency;
    for (const auto& combo : combinations) {
        long long secret = lagrangeInterpolation(combo);
        secretFrequency[secret]++;
    }

    // Find the most common secret
    long long finalSecret = -1;
    int maxFreq = 0;
    for (const auto& [secret, freq] : secretFrequency) {
        if (freq > maxFreq) {
            maxFreq = freq;
            finalSecret = secret;
        }
    }

    cout << "Final Secret: " << finalSecret << endl;
    return 0;
}

