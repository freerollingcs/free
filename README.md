# free
 @RequestMapping("/confirmAdd")
    public String confirmAdd(@Valid @ModelAttribute(SESSION_KEY_SEARCH_FORM_NAME) BlackListForm form,
            BindingResult result, Model model, HttpSession session) throws PEException {
        model.addAttribute("memberLogonTypeMap", MemberLogonTypeEnum.getMap());
        model.addAttribute("remarkTypeMap", RemarkTypeEnum.getMap());
        if (result.hasErrors()) {
            model.addAttribute("validation", false);
            return "blackList/add";
        }
        Blacklist blackList = form.getBlackList();
        String custNum = null;
        // 当有卡号的时候需要查询出来会员编号入库
        if (StringUtils.isNotEmpty(blackList.getSource())) {

            try {
                custNum = remoteService.queryCustNum(blackList.getSource());
            } catch (PEException e) {
                LOGGER.error(e.getMessage(), e);
                result.reject(null, "没有[" + blackList.getSource() + "]对应的会员编号。");
                model.addAttribute("validation", false);

                return "blackList/add";
            }
            if (StringUtils.isEmpty(custNum)) {
                result.reject(null, "没有[" + blackList.getSource() + "]对应的会员编号。");
                model.addAttribute("validation", false);

                return "blackList/add";
            }
            blackList.setCustNum(custNum);
        }

        if (null != blacklistService.selectByCustNum(blackList.getCustNum())) {
            if (StringUtils.isNotEmpty(custNum)) {
                result.reject(null, "黑名单卡号" + blackList.getSource() + "]已经存在。");
                model.addAttribute("validation", false);
            } else {
                result.reject(null, "黑名单会员编号[" + blackList.getCustNum() + "]已经存在。");
                model.addAttribute("validation", false);
            }
            return "blackList/add";
        }

        session.setAttribute(SESSION_KEY_SEARCH_FORM_NAME, form);
        return "blackList/confirmAdd";
    }

    /**
     * 数据入库
     */
    @RequestMapping("/saveAdd")
    public String saveAdd(HttpSession session, HttpServletRequest request) throws PEException {
        BlackListForm form = (BlackListForm) session.getAttribute(SESSION_KEY_SEARCH_FORM_NAME);
        Blacklist blackList = form.getBlackList();
        BaseParamUtil.setBaseObjectForCreate(blackList, request);
        blackList.setAllEmptyStringToNull();
        blackList.trimAllString();
        // 插入黑名单
        blacklistService.insert(blackList);

        session.removeAttribute(SESSION_KEY_SEARCH_FORM_NAME);

        return "redirect:/blackList/summary?keepSp=Y";
    }
