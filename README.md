### Grammatica secondo livello 

Aggiunti gli statements if e for


### Grammatica terzo livello

Aggiunti gli operatori di algebra booleana AND, OR e NOT
Per farlo ho aggiunto i token e ho modificato la classe BinaryExprAST:

    
Value *BinaryExprAST::codegen(driver& drv) {
  if(Op == 'n') {
    //Not
    Value *L = LHS->codegen(drv);
    if (!L) return nullptr;
    else return builder->CreateNot(L,"notres");
  }
  Value *L = LHS->codegen(drv);
  Value *R = RHS->codegen(drv);
  if (!L || !R) 
     return nullptr;
  switch (Op) {
  case '+':
    return builder->CreateFAdd(L,R,"addres");
  case '-':
    return builder->CreateFSub(L,R,"subres");
  case '*':
    return builder->CreateFMul(L,R,"mulres");
  case '/':
    return builder->CreateFDiv(L,R,"addres");
  case '<':
    return builder->CreateFCmpULT(L,R,"lttest");
  case '=':
    return builder->CreateFCmpUEQ(L,R,"eqtest");
  case 'a':
    //And
    return builder->CreateLogicalAnd(L,R,"andres");
  case 'o':
    //Or
    return builder->CreateLogicalOr(L,R,"orres");
  default:  
    std::cout << Op << std::endl;
    return LogErrorV("Operatore binario non supportato");
  }
};