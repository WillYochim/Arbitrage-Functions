//The remainder of the code is private


// Simple Manual Calculation For Amount Out Which Replaces The Use of the Contract Call or getAmountOutQuickswap(token1, token2, amount)
function manualCalculateAmountOut(amountIn, reserveIn, reserveOut) {
    const swapRate = Number(reserveIn) / Number(reserveOut);
    //console.log(swapRate);
    const amountInWithFee = Number(amountIn) * 997;
    const numerator = amountInWithFee * Number(reserveOut);
    const denominator = Number(reserveIn) * 1000 + amountInWithFee;
    let amountOut = numerator / denominator;
    return Number(amountOut);
}

// Calculate Price Impact Using Market Price and Price After Estimated Swap Impact
function calculatePriceImpact(currentPrice, amountIn, reserveIn, reserveOut) {
    const amountOut = manualCalculateAmountOut(amountIn, reserveIn, reserveOut);
    const postPrice = amountIn / amountOut;
    console.log(postPrice);
    const priceImpact = ((postPrice / currentPrice) - 1) * 100;
    return priceImpact;
}

// Binary Search to Find Optimal Amount
function calculateMaxSwapAmount(currentPrice, reserveIn, reserveOut, maxPriceImpactPercentage) {
    let left = 0;
    let right = reserveIn; 
    let maxAmountIn = 0;

    while (left <= right) {
        const mid = (left + right) / 2;
        const priceImpact = calculatePriceImpact(currentPrice, mid, reserveIn, reserveOut);

        if (priceImpact <= maxPriceImpactPercentage) {
            maxAmountIn = mid;
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    const finalPriceImpact = calculatePriceImpact(currentPrice, maxAmountIn, reserveIn, reserveOut);
    console.log(`Price Impact: ${finalPriceImpact}%`);
    return maxAmountIn;
}

async function getPriceParaswap(srcToken, destToken, amount, maxImpact) {
    const url = 'https://apiv5.paraswap.io/prices?';

    // Initialize with the initial amount
    let currentAmount = BigInt(amount * 10 ** polygonTokens[srcToken].decimals);
    
    const params = new URLSearchParams({
        srcToken: polygonTokens[srcToken].address,
        srcDecimals: polygonTokens[srcToken].decimals.toString(),
        destToken: polygonTokens[destToken].address,
        destDecimals: polygonTokens[destToken].decimals.toString(),
        amount: currentAmount.toString(),
        side: 'SELL',
        network: '137',
        maxImpact: maxImpact.toString()
    });

    const response = await fetch(`${url}${params.toString()}`);
    const data = await response.json();

    // Extract parsed values for calculations
    const destUSD = parseFloat(data.priceRoute.destUSD);
    const srcUSD = parseFloat(data.priceRoute.srcUSD);

    // Perform the calculations
    const difference = 1 - (destUSD / srcUSD);
    const priceImpact = 100 * difference;

    //console.log(`Amount: ${currentAmount.toString()}, Impact Loss: ${priceImpact}`);

    // Log the response for debugging
    //console.log("API Response:", data);
    return {
        price: data.priceRoute.destAmount,
        impact: priceImpact,
        payload: data.priceRoute
    };
    //return getTokenInDecimalPolygon(destToken, data.priceRoute.destAmount);
}


//Define Slippage as Global
// const slippage = 0.00;
async function buildTransactionParaswap(pricePayload, srcToken, destToken, srcAmount, minDestAmount, network, userAddress) {

    srcAmount = BigInt(srcAmount * 10 ** polygonTokens[srcToken].decimals);
    try {
        const requestURL = `${'https://apiv5.paraswap.io'}/transactions/${network}`;
        const requestData = {
            priceRoute: pricePayload,
            srcToken: polygonTokens[srcToken].address,
            destToken: polygonTokens[destToken].address,
            srcAmount: srcAmount.toString(),
            destAmount: minDestAmount,
            userAddress: userAddress,
            srcDecimals: polygonTokens[srcToken].decimals.toString(),
            destDecimals: polygonTokens[destToken].decimals.toString(),
        };

        const response = await fetch(requestURL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(requestData),
        });
        const data = await response.json();
        console.log(data);
        return {
            from: data.from,
            to: data.to,
            data: data.data,
            gasLimit: '0x' + Number(data.gas).toString(16),
            value: '0x' + Number(data.value).toString(16),
        };
    } catch (e) {
        throw new Error( `Paraswap unable to buildTransaction ${polygonTokens[srcToken].address} ${polygonTokens[destToken].address} ${network} ${e.message}`);
    }
}

async function executeTx(txRequest) {
    const tx = await signer.sendTransaction(txRequest);
    return await tx.wait(); 
}

async function calcAmountToTradeTests(){
    console.log("Timestamp: " + new Date().toISOString());
    console.time("myFunction");
    const pricing = await getPriceParaswap("NAKA", "USDC", 10, 1);
    const transaction = await buildTransactionParaswap(pricing.payload, "NAKA", "USDC", "10", pricing.price, "137", "");
    console.log(transaction);
    let executedTransaction = await executeTx(transaction);
    console.log(executedTransaction);
    //console.log(pricing);
    //Check for arb possibility
    //if arb execute arb:

    console.log(pricing);
    console.timeEnd("myFunction");
    console.log("Timestamp: " + new Date().toISOString());
}
