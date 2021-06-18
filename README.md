# cloudbuffer

### 改动1
删掉txn.cpp/Commit函数中只读事务的ValidateOCC调用，现在只需要调用ValidayeReadInMerge即可。
### 改动2
修改ValidateInMerge函数，改成当一个事务跨epoch时即commit_epoch != start_epoch时才需要验证读集，否则直接返回true即可，改动如下：
bool OccTransactionManager::ValidateReadInMerge(TxnManager * txMan){
    // MOT_LOG_INFO("验证开始在merge期间执行的事务");

    auto start_epoch = txMan->GetStartEpoch();
    auto commit_epoch = txMan->GetCommitEpoch();
    if(commit_epoch == start_epoch) return true; // 不是跨epoch的数据，不用验证，直接返回true即可。
    
    TxnOrderedSet_t &orderedSet = txMan->m_accessMgr->GetOrderedRowSet();
    bool result = true;
    for (const auto &raPair : orderedSet)
    {
        const Access *ac = raPair.second;
        if (ac->m_type == RD)
        {
            if (!ac->GetRowFromHeader()->m_rowHeader.ValidateRead(ac->m_cts))
            {
                return false;
            }
        }
    }
    //MOT_LOG_INFO("长事务未与当前epoch冲突");
    return result;
}
