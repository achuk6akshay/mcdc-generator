(function($, window, undefined) {
    var that = {};
    window.truth = that;

    var SYMBOL = /[a-zA-Z]\w*/;
    var WHITESPACE = /^\s*$/;
    var DEBUG = true;
	var trueArray =[];
	var falseArray = [];
	var finalArr = [];
	var duplicateVariable = 0;

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
			var finalOutput = getConstrainedCases(trueArray,falseArray,sym);
			var testCases = finalOutput[0];
			var variableArray = finalOutput[1];
			displayConstrainedCases(testCases,val,sym,variableArray);

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
			if (left  || right){
				//debug(' ANDevalsymmA happening',evalSym(1));
				  //debug('ANDevalsymmB happening',evalSym(2));
				 // debug('-----------------------------------------------------------');
				return resu1;
			} 
			else {
				//debug(' ANDevalsymmA',evalSym(1));
				  //debug('ANDevalsymmB',evalSym(2));
				 // debug('-----------------------------------------------------------'); 

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
			if (left  && right) return  null;
			else return resu;

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
			if(symbol in symbols){
				duplicateVariable++;
			}
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
		
	function getConstrainedCases(trueArray,falseArray,symbols){
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
					var breakLimit = 0;
					//debug("trueArray.length + falseArray.length",trueArray.length + falseArray.length);
					while ((testCasesCount < (conArray[0].length) )&& (breakLimit <= (trueArray.length + falseArray.length))){
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
					breakLimit++;
				    debug("breakLimit", breakLimit);
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
					var breakLimit = 0;
					debug("trueArray.length + falseArray.length",trueArray.length + falseArray.length);
					while (testCasesCount < (conArray[0].length)&& (breakLimit <= (trueArray.length + falseArray.length))){
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
					
				breakLimit++;
				debug("breakLimit", breakLimit);
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
	
	function displayConstrainedCases(a,expression,sym,variableArray){
		 $('#final').empty();

        var ret = $('<table border="1"/>');

        var header = $('<tr>');
        
        $.each(sym, function(ind) {
            header.append($('<th>').text(ind));
        });
		header.append($('<th>').text(expression));
		header.append($('<th>').text("variable"));
        ret.append(header);
        $('#final').append(ret);
		
		for(i in a){
			var comboRow = $('<tr>');
			for (j in a[i]){
			    //debug("a[i][j]",a[i][j]);
			    comboRow.append($('<td>').text(a[i][j]));
			}
			
			
			if (i > 0){
				var counter = 0;
			    $.each(sym, function(ind) {
                
				if (counter == variableArray[i-1] ){
					comboRow.append($('<td>').text(ind));
				}
				counter++;
                });
			}
		ret.append(comboRow);
		$("#final").append(ret);
		}
		
	}
	
})(jQuery, window);


