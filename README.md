(function($, window, undefined) {
    var that = {};
    window.truth = that;

    var SYMBOL = /[a-zA-Z]\w*/;
    var WHITESPACE = /^\s*$/;
    var DEBUG = true;
	var trueArray =[];
	var falseArray = [];
	var finalArr = [];

    function truthCombos(symbols) {
        if (! symbols) {
            return [{}];
        }

        var key;
        // get a key, any key
        $.each(symbols, function(k, v) {
            key = k;
            return false;
        });
        if (! key) {
            return [{}];
        }

        var tmp = jQuery.extend({}, symbols);
        delete tmp[key];
        var prev = truthCombos(tmp);

        var ret = [];
        $.each(prev, function(i) {
            var cur = jQuery.extend({}, prev[i]);
            cur[key] = true;
            ret.push(cur);
            cur = jQuery.extend({}, prev[i]);
            cur[key] = false;
            ret.push(cur);
        });

        return ret;
    }
    that.truthCombos = truthCombos;

    function displayAST(ast) {
        function elemGen(ast) {
            //debug('displayAST', ast);
            if (! $.isArray(ast) ) {
				debug('displayRet', ast);
                return $("<li>").text(ast);
            }

            var ret = $('<ul>');
            $.each(ast, function(i, val) {
                ret.append(elemGen(val));
				
            });
			debug('displayRet', ret);
            return ret;
        }

        $('#ast').empty().append('<h1>Abstract Syntax Tree</h1>', elemGen(ast));
    }

    function handleInput(val) {
        try {
            $('#nothing').empty();
            var tok = tokenize(val);
            var results = parse(tok);
            var ast = results[0];			
			
            var sym = results[1];
            debug('ast', ast);
            debug('sym', sym);
            displayAST(ast);
            displaySym(sym);
			//debug('sezeOf',sizeOfAst(ast));
            displayCombos(val, sym, ast, truthCombos(sym));
			displayMcdc(val, sym, ast, truthCombos(sym));
			debug('trueArray',trueArray);
			debug('falseArray',falseArray);
			var testCases = getConstrainedCases(trueArray,falseArray)[0];
			displayConstrainedCases(testCases,val,sym);
			/* $('#final').append('<table border="1"/>');
			for (i in finalArr)
			{
				debug("greeet");
				var com = $('<tr>');;
				com.append($('<td>').text(finalArr[i]));
				$('#final').append(com);
				$('#final').append('</tr>');
				
			}  */
			/* depth_return = f(ast,0);
			console.log("####################### Deepest Operation : "+depth_return[0]);
			console.log("####################### Depth : "+depth_return[1]); */
        } catch (e) {
            out('Unable to parse expression. ' + e);
            throw e;
        }
    }

    function displayCombos(expression, symbols, ast, combos) {
        $('#combo').empty();
        //debug(combos);

        var ret = $('<table border="1"/>');

        var header = $('<tr>');
        var symArr = [];
        $.each(symbols, function(sym) {
			
            header.append($('<th>').text(sym));
            symArr.push(sym);
        });
        header.append($('<th>').text(expression));

        ret.append(header);
        $('#combo').append(ret);

        $.each(combos, function(i) {
            var cur = combos[i];
            var result = evalExpr(ast, cur);
            //debug('evaluated', cur, result);
            var comboRow = $('<tr>');
            $.each(symArr, function(j) {
				//debug('current now', cur[symArr[j]]);
                comboRow.append($('<th>').text(cur[symArr[j]]));
            });
            comboRow.append($('<td>').text(result));
            ret.append(comboRow);
        });
    }
	
	 function displayMcdc(expression, symbols, ast, combos) {
        $('#mcdc').empty();
        //debug(combos);

        var ret = $('<table border="1"/>');

        var header = $('<tr>');
        var symArr = [];
        $.each(symbols, function(sym) {
            header.append($('<th>').text(sym));
            symArr.push(sym);
        });
        header.append($('<th>').text(expression));

        ret.append(header);
        $('#mcdc').append(ret);

        $.each(combos, function(i) {
            var cur = combos[i];
            var result = evalExpr1(ast, cur);
            debug('evaluated', cur, result);
			if(result !== null){
				if(result){
					var a = [];
					$.each(cur, function(key, value) {
                    a.push(value);
                });
					a.push(result);
					trueArray.push(a);
				}
				else {
					var a = [];
					$.each(cur, function(key, value) {
                    a.push(value);
                });
					a.push(result);
					falseArray.push(a);
					
				}
				
            var comboRow = $('<tr>');
            $.each(symArr, function(j) {
                comboRow.append($('<td>').text(cur[symArr[j]]));
            });
            comboRow.append($('<td>').text(result));
            ret.append(comboRow);
			}
        });
    }


    function isBoolean(val) {
        return (val === true) || (val === false);
    }

    function assertBoolean(val) {
        if (! isBoolean(val)) {
            throw new SyntaxError('Unbound symbol: ' + val);
        }
    }
			var count=0;

    function evalExpr1(ast, bindings) {
        function evalSym(index) {
            if ($.isArray(ast[index])) {
                return evalExpr1(ast[index], bindings);
            }

            assertBoolean(bindings[ast[index]]);
            return bindings[ast[index]];
        }

        if (! ast) {
            throw new SyntaxError('Invalid expression: ' + ast);
        }

        if (! $.isArray(ast)) {
            return bindings[ast];
        }

        switch(ast[0]) {
        case '&': 
		    /* if(!(evalSym(1) || evalSym(2))){
				debug(' ANDevalsymmA',evalSym(1));
				  debug('ANDevalsymmB',evalSym(2));
				  debug('-----------------------------------------------------------');
				return null;
			}
            else {
				  debug(' ANDevalsymmA happening',evalSym(1));
				  debug('ANDevalsymmB happening',evalSym(2));
				  debug('-----------------------------------------------------------');
				  return  evalSym(1) && evalSym(2);
			} */
			var left = evalSym(1);
			var right = evalSym(2);
		
			var resu1 =  left  && right;
			if ((left  || right)===true){
				debug(' ANDevalsymmA happening',evalSym(1));
				  debug('ANDevalsymmB happening',evalSym(2));
				  debug('-----------------------------------------------------------');
				return resu1;
			} 
			else {
				debug(' ANDevalsymmA',evalSym(1));
				  debug('ANDevalsymmB',evalSym(2));
				  debug('-----------------------------------------------------------'); 

				  return  null;
			}
        case '^':
            return (evalSym(1) || evalSym(2)) &&
                   (!(evalSym(1) && evalSym(2)));
        case '|':
            /* if(!(evalSym(1) && evalSym(2))){
				debug(' ORevalsymmA happening',evalSym(1));
				  debug('ORevalsymmB happening',evalSym(2));
				  debug('-----------------------------------------------------------');
				return evalSym(1) || evalSym(2);
			}
            else {
				  debug(' ORevalsymmA',evalSym(1));
				  debug('ORevalsymmB',evalSym(2));
				  debug('-----------------------------------------------------------');
				  return null;
			} */
			var left = evalSym(1);
			var right = evalSym(2);
			var resu =   left || right;
			if ((left  && right)===false) return  resu;
			else {
				//storingIndexValue.push(count);
				return null;
			}

				case '!':
        case '~':
            return ! evalSym(1);
        default:
            throw new SyntaxError('Unrecognized operator: ' + ast[0]);
			
							  storingIndexValue.push(count);
				  count++;
				  debug('Index not required',storingIndexValue);
        }
		
    }
	//console.log('indexe not req'+storingIndexValue);
			
    that.evalExpr1 = evalExpr1;
	
	function evalExpr(ast, bindings) {
        function evalSym(index) {
            if ($.isArray(ast[index])) {
                return evalExpr(ast[index], bindings);
            }

            assertBoolean(bindings[ast[index]]);
            return bindings[ast[index]];
        }

        if (! ast) {
            throw new SyntaxError('Invalid expression: ' + ast);
        }

        if (! $.isArray(ast)) {
            return bindings[ast];
        }

        switch(ast[0]) {
        case '&':
            return evalSym(1) && evalSym(2);
        case '^':
            return (evalSym(1) || evalSym(2)) &&
                   (!(evalSym(1) && evalSym(2)));
        case '|':
            return evalSym(1) || evalSym(2);
        case '!':
        case '~':
            return ! evalSym(1);
        default:
            throw new SyntaxError('Unrecognized operator: ' + ast[0]);
        }
    }
    that.evalExpr = evalExpr;

    function out(val) {
        $('#nothing').text(val.toString());
        $('#ast').empty();
        $('#combo').empty();
        $('#sym').empty();
    }

    function displaySym(symbols) {
        var title = $('<h1>Symbols</h1>');
        var symTable = $('<ol/>');
        $.each(symbols, function(sym) {
            symTable.append($('<li/>').text(sym));
        });
        $('#sym').empty().append(title, symTable);
    }

    function main() {
        var self = this;
        $('#expr').keyup(function() {
            //debug('keyup');
			trueArray = [];
			falseArray = [];
            var val = $('#expr').val();
            if (this.lastSearch !== val) {
                this.lastSearch = val;
                handleInput($('#expr').val());
            }
        });

        $('#expr').val('a&b|c');
        $('#expr').keyup();
    }
    that.main = main;

    /**
     * Build an abstract syntax tree out of the given stream of tokens.
     */
    var parse = function() {
        var pos;
        var symbols;
        var tokens;

        function getCurToken() {
            return tokens[pos];
        }

        /**
         * Consume the next token and add it to the symbol table.
         */
        function consumeSymbol() {
            //debug('consumeSymbol');
            var symbol = consumeToken();
            symbols[symbol] = true;
            return symbol;
        }

        function consumeToken(expected) {
            var curTok = getCurToken();
            //debug('consumeToken', curTok, pos);
            if (expected && (curTok !== expected)) {
                throw new SyntaxError('Did not encounter expected token. Expected: "' +
                                      expected + '" Actual: "' + curTok + '".');
            }

            pos++;
            return curTok;
        }

        function expr() {
            //debug('expr', tokens, pos);
            if ((getCurToken() === '!') ||
                (getCurToken() === '~')) {
                return [consumeToken(), binaryExpr()];
            }

            return binaryExpr();
        }

        function binaryExpr() {
            return xorExpr();
        }

        function xorExpr() {
            var a1 = orExpr();
            if (getCurToken() === '^') {
                return [consumeToken(), a1, xorExpr()];
            }

            return a1;
        }

        function orExpr() {
            var a1 = andExpr();
            if (getCurToken() === '|') {
                return [consumeToken(), a1, orExpr()];
            }

            return a1;
        }

        function andExpr() {
            var a1 = subExpr();
            if (getCurToken() === '&') {
                return [consumeToken(), a1, andExpr()];
            }

            return a1;
        }

        function subExpr() {
            //debug('subExpr', tokens, pos);
            if (getCurToken() === '(') {
                consumeToken('(');
                var ret = expr();
                consumeToken(')');
                return ret;
            }

            if (SYMBOL.test(getCurToken())) {
                return consumeSymbol();
            }

            return expr();
        }
        
        return function(tok) {
            //debug('parse', tok);
            if ((! tok) || (! tok.length)) {
                return [];
            }

            tokens = tok;
            pos = 0;
            symbols = {};
            var ret = expr();
            if (pos < tokens.length) {
                //debug(tokens);
                //debug(pos);
                //debug(tokens[pos]);
                throw new SyntaxError('Could not consume all tokens. ' +
                                      'Remaining tokens: ' +
                                      tokens.slice(pos, tokens.length));
            }

            return [ret, symbols];
        };
    }();
    that.parse = parse;


    function tokenize(str) {
        if ((! str) || WHITESPACE.test(str)) {
            return [];
        }

        var ret = str.split(/\b/);
        for (var i = 0; i < ret.length; i++) {
            // Remove whitespace from tokens
            ret[i] = ret[i].replace(/\s/g, '');

            if (! ret[i].length) {
                // Delete empty element
                ret.splice(i, 1);
            } else if (! SYMBOL.test(ret[i])) {
                var arr = [];
                // For consecutive non-symbol characters,
                // split each character into individual tokens
                for (var j = 0; j < ret[i].length; j++) {
                    arr.push(ret[i][j]);
                }

                // Replace element with all subtokens of
                // current element
                Array.prototype.splice.apply(ret, [i, 1].concat(arr));
            }
        }

        return ret;
    }
    that.tokenize = tokenize;
	
	function sizeOfAst(ast) {
		var clone = arrayClone(ast);
		function sizeFinder(clone){
		  for (i in clone) {
			  if ($.isArray(clone[i])){
				  sizeFinder(clone[i]);
			  }
			  else if (clone[i] === parseInt(clone[i], 10)) {
				  debug('hello');
				  clone[i]=clone[i]+1;
			  }
			  else {clone[i] = 1;}
		  }
		}
		sizeFinder(clone);
      debug('clone',clone);
	  debug('ast',ast);
	  //return clone;		
	}
	
	function arrayClone( arr ) {

    var i, copy;

    if( Array.isArray( arr ) ) {
        copy = arr.slice( 0 );
        for( i = 0; i < copy.length; i++ ) {
            copy[ i ] = arrayClone( copy[ i ] );
        }
        return copy;
    } else if( typeof arr === 'object' ) {
        throw 'Cannot clone array containing an object!';
    } else {
        return arr;
    }

}

    function debug() {
        if (DEBUG && window.console && window.console.log) {
            console.log.apply(console, arguments);
        }
    }
	
	/* var f = function(arrx,depthx,sidex	){
		let arr = arrx;
		let depth = depthx;
		let side = sidex;
		if(side==undefined|side==null){
			side = "Right";
		}
		
			console.log("Inside "+side+" function ");
	
		
		console.log("Array"+arr);
		console.log("Array Length "+arr.length);
		console.log("depth"+depth);
	if(arr.length==3){
		operator = arr[0];
	let	left=arr[1];
	let	right=arr[2];
		console.log("Left : "+left);
		console.log("Right : "+right);

		let	left_return;
		if(!$.isArray(left))		
		left_return = [arr,depth];
		else{
		left_return = 	f(left,depth+1,"Left");
		
		}
		let	right_return;
		if(!$.isArray(right))
		right_return = [arr,depth];
		else{
						

		right_return = 	f(right,depth+1);

		}

		if(left_return[1]>right_return[1]){
			console.log("left was returned : Left return: "+left_return+"\nRight return : "+right_return);
			return left_return;
		}else{
			console.log("right was returned : Left return : "+left_return+"\nRight return : "+right_return);
			return right_return;
		}
		
	}else{
		
	let	operator = arr[0];
	let	right = arr[1];
	console.log("Inside Not : Right : "+right);
		if(!$.isArray(right)){
			console.log("constant");
			return [arr,depth];
		}
		else{
			return 	f(right,depth+1);
		}
		
	}
} */

    /* function displayConstrainedCases(trueArray,falseArray){
		var midArr = [];
		if (trueArray.length>falseArray.length) {
			console.log('inside true greater');
			for(var i in falseArray){
				
				for(var j in trueArray){
					var counter = 0;
					var resMisMatch = false;
					for(var z in trueArray[0]){
						
                        if(z < (trueArray[0].length-1)){
						    //console.log('yes');
					        if(trueArray[j][z] === falseArray[i][z]){
							    
							    counter++;
								//console.log(counter);
								
						    }
					    }
						else{
							if(trueArray[j][z] !== falseArray[i][z]){
								//debug('resmismatch');
								resMisMatch = true;
							}
						}
					}
					if (counter === falseArray[0].length-2 && resMisMatch){
					    midArr.push(trueArray[j]);
						debug('test case',falseArray[i],trueArray[j]);
				    }
				}
				debug('midArr',midArr);
				finalArr.push(midArr);
				midArr = [];
				debug('finalArr',finalArr);
				
				//counter=0;
			}
		}
		else {
			console.log('inside false greater');
			for(var i in trueArray){
				
				for(var j in falseArray){
					var counter = 0;
					var resMisMatch = false;
					for(var z in trueArray[0]){
						
                        if(z < (trueArray[0].length-1)){
						    //console.log('yes');
					        if(trueArray[i][z] === falseArray[j][z]){
							    
							    counter++;
								//console.log(counter);
								
						    }
					    }
						else{
							if(trueArray[i][z] !== falseArray[j][z]){
								//debug('resmismatch');
								resMisMatch = true;
							}
						}
					}
					if (counter === falseArray[0].length-2 && resMisMatch){
					    midArr.push(falseArray[j]);
						debug('test case',trueArray[i],falseArray[j]);
				    }
				}
				debug('midArr',midArr);
				finalArr.push(midArr);
				debug('finalArr',finalArr);
				
				//counter=0;
			}
		}
	} */
	
/* 	 function displayConstrainedCases(trueArray,falseArray){
		var conArray = [];
		if (trueArray.length>falseArray.length) {
		    console.log('inside true greater');
		}
		else{
			console.log('inside false greater');
			debug("length",trueArray[0].length);
			
				//debug("k value",k);
				if (trueArray.length == 1){
					
				}
				
			    else{
					for(i in trueArray){
						for(j in falseArray){
							var counter = 0;
							for(k = 0; k < trueArray[0].length-1;k++){
								if (trueArray[i][k] != falseArray[j][k]){
									counter++;
								}
							}
							if (counter == 1){
								var trueFlag = true;
								var falseFlag = true;
								for(y in conArray){
									if (conArray[y].toString() === trueArray[i].toString()){
										//debug("matching value",trueArray[i].toString(),conArray[y].toString());
										trueFlag = false;
									}
									if (conArray[y].toString() === falseArray[j].toString()){
										//debug("matching value",falseArray[j].toString(),conArray[y].toString());
										falseFlag = false;
									}
								}
										
								if (trueFlag){
									debug("value of z",);
									conArray.push(trueArray[i]);
									debug("contrueArray",trueArray[i]);
								}
								if(falseFlag){
									conArray.push(falseArray[j]);
									debug("contrueArray",trueArray[i]);
									debug("confalseArray",falseArray[j]);
								}
							}
						}
				}
				debug("conArray",conArray);
		    }
	    } */
		
	function getConstrainedCases(trueArray,falseArray){
		var conArray = [];
		var varArray = [];
		if (trueArray.length>falseArray.length) {
		    console.log('inside true greater');
			debug("length",trueArray[0].length-1);
					var varArray = [];
					var flag = [];
					primaryLoop:
					for(i in falseArray){
						for(j in trueArray){
							flag = mcdcEqualCheck(trueArray[j],falseArray[i]);
							if (flag[0]){
								conArray.push(falseArray[i]);
								conArray.push(trueArray[j]);
								debug("con Array primary", conArray);
								varArray.push(flag[1]);
							    break primaryLoop;
							}
						}
					}
					var testCasesCount = conArray.length;
					
					while (testCasesCount < conArray[0].length){
					for (k in conArray){
						//debug("testCasesCount",testCasesCount);
						if(conArray[k][conArray[k].length-1]){
							
							//debug("inside true conArray check",conArray[k]);
							for (z in falseArray){
								flag = mcdcEqualCheck(conArray[k],falseArray[z]);
								if (flag[0] && !(varArray.includes(flag[1]))){
									//debug("mcdc success",conArray[k],falseArray[z],flag[1]);
								    conArray.push(falseArray[z]);
									testCasesCount++;
								    varArray.push(flag[1]);
									
							    }
							} 
						}
						else {
							//debug("inside false conArray check",conArray[k]);
							for (z in trueArray){
								flag = mcdcEqualCheck(conArray[k],trueArray[z]);
								if (flag[0] && !(varArray.includes(flag[1]))){
									//debug("mcdc success",conArray[k],trueArray[z],flag[1]);
								    conArray.push(trueArray[z]);
									testCasesCount++;
								    varArray.push(flag[1]);
									
							    }
							} 
						}
					}
					
				
				}
				
		}
		else{
			console.log('inside false greater');
			debug("length",trueArray[0].length-1);
			
				//debug("k value",k);

					
					var flag = [];
					primaryLoop:
					for(i in trueArray){
						for(j in falseArray){
							flag = mcdcEqualCheck(trueArray[i],falseArray[j]);
							if (flag[0]){
								conArray.push(trueArray[i]);
								conArray.push(falseArray[j]);
								debug("con Array primary", conArray);
								varArray.push(flag[1]);
								debug("greeeeet");
							    break primaryLoop;
							}
						}
					}
					var testCasesCount = conArray.length;
					
					while (testCasesCount < conArray[0].length){
					for (k in conArray){
						//debug("testCasesCount",testCasesCount);
						if(conArray[k][conArray[k].length-1]){
							
							//debug("inside true conArray check",conArray[k]);
							for (z in falseArray){
								flag = mcdcEqualCheck(conArray[k],falseArray[z]);
								if (flag[0] && !(varArray.includes(flag[1]))){
									//debug("mcdc success",conArray[k],falseArray[z],flag[1]);
								    conArray.push(falseArray[z]);
									testCasesCount++;
								    varArray.push(flag[1]);
									
							    }
							} 
						}
						else {
							//debug("inside false conArray check",conArray[k]);
							for (z in trueArray){
								flag = mcdcEqualCheck(conArray[k],trueArray[z]);
								if (flag[0] && !(varArray.includes(flag[1]))){
									//debug("mcdc success",conArray[k],trueArray[z],flag[1]);
								    conArray.push(trueArray[z]);
									testCasesCount++;
								    varArray.push(flag[1]);
									
							    }
							} 
						}
					}
					
				
				}
				
		}
		debug("conArray",conArray);
		debug("var Arr",varArray);
		return [conArray,varArray];
	}
	
	function mcdcEqualCheck(a,b){
		var counter = 0;
		var index = 0;
		for(let k = 0; k < a.length-1;k++){
			if (a[k] != b[k]){
				counter++;
				index = k;
			}
		}
		if (counter == 1){	
		    debug("inside mcdc check",a,b,index);
            return [true,index];		
		}
		return [false,0]
	}
	
	function displayConstrainedCases(a,expression,sym){
		 $('#final').empty();

        var ret = $('<table border="1"/>');

        var header = $('<tr>');
        
        $.each(sym, function(ind) {
            header.append($('<th>').text(ind));
        });
		header.append($('<th>').text(expression));
        ret.append(header);
        $('#final').append(ret);
		
		for(i in a){
			var comboRow = $('<tr>');
			for (j in a[i]){
			    //debug("a[i][j]",a[i][j]);
			    comboRow.append($('<td>').text(a[i][j]));
			}
		ret.append(comboRow);
		$("#final").append(ret);
		}
		
	}
	
})(jQuery, window);

